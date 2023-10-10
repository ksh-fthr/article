Gradle を使ってデバッグ実行する際に掲題のエラーがでて、その対処法について調べ方や対処の仕方がまずかったせいで一日ハマった。
で、今後同じ現象が発生したときに悩む時間や調べる時間が勿体無いので備忘録として残す。

もう本当の意味での備忘録で、なぜこうすることで対処できたのか、とかの検証やドキュメントの確認とかはしていない。


## 環境
* Windows7 64bit
* Java 8
* Gradle(バージョンは忘れた)

## 現象
前掲の通り。デバッグのためにビルド～デバッグ実行を Windows 環境で行ったところ、

```:例外メッセージ
... 前略 ...
java.io.IOException: CreateProcess error=206, ファイル名または拡張子が長すぎます。
... 後略 ...
```

が発生し、アプリが起動しない。
原因は例外メッセージのまんまで、Windows 環境で実行時に巻き込んでいるライブラリやらの階層が深すぎたりすると発生するらしい。

## 対処
次のコードを build.gradle に追加したらエラーは発生しなくなり、デバッグ実行が無事行えるようになった。

```:build.gradle
... 前略 ...
task pathingJar(type: Jar) {
    dependsOn configurations.runtime
    appendix = 'pathing'
 
    doFirst {
        manifest {
            attributes "Class-Path": configurations.runtime.files.collect {
                it.toURL().toString().replaceFirst(/file:\/+/, '/')
            }.join(' ')
        }
    }
}
 
bootRun {
    ... 前略 ...
    dependsOn pathingJar
    doFirst {
        classpath = files("$buildDir/classes/main", "$buildDir/resources/main", pathingJar.archivePath)
    }
    ... 後略 ...
}
... 後略 ...
```

### 参考
https://github.com/grails/grails-core/issues/9125

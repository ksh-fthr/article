Spring Boot でタスクを定期実行するには @Scheduled を指定するのが手軽で良い。
だが @Scheduled のデフォルトスレッドプールが**シングルスレッド**ということを知らないと、思わぬ落とし穴にハマるのでご注意を。

## ハマったケース

次のような構成でプールサイズを意識しないで実装すると、ClassB の処理でスレッドが掴まれたままで ClassA の処理が行われなくなる。

### 構成

ClassA.java というクラスで「毎分 0秒 と 30秒 に処理を行う」設定をし

```java:ClassA.java
@EnableScheduling
ClassA {

    @Scheduled(cron = "0,30 * * * * *")
    public void execute() {
        // todo
    }
}
```

ClassB.java というクラスで「起動後 60秒 経ったら処理を行い、その処理が終了した後に 30秒 経ったら次の処理を行う」という設定をした。

```java:ClassB.java
@EnableScheduling
ClassB {

    @Scheduled(initialDeley = 1000 * 60, fixedDelay = 1000 * 30)
    public void execute() {
        // 他の処理の結果を受けて処理を終了するポーリング
    }
}
```

### スレッドが掴まれる状況

```:スレッドが掴まれる可能性のあるケース
                  16:00:00          16:00:30           16:01:00
ClassA.java |---------+-----------------+-----------------+------------>
            |
            |                               起動後 60秒
ClassB.java |-------------------------------------+-------------------->
```

この図の説明をすると

* ClassA では 16:00:00、16:00:30、16:01:00 と、30秒 毎に処理が定期的に行われる
* ClassB では起動後 60秒 経過してから最初の処理が行われる

ことを想定したものである。
で、

* ClassB の処理が (ここでは登場しない) 他の処理の結果を受けて処理を終了するポーリング処理で、延々と処理をし続ける

ことになった場合、

* ClassA は 16:01:00 の処理がいつまでたっても開始されない

状態に陥る。これは

* @Scheduled のデフォルトスレッドプールが**シングルスレッド**

であることが原因で、**ClassB のポーリング処理がスレッドを掴みっぱなしになっている**ために発生する。

このケースで厄介なのは、ClassA の方では初回の 16:00:00 と 16:00:30 の処理がしっかり行われること。
デバッグ実行でブレイクポイントを貼ったとき、16:01:00 の処理からブレイクポイントに止まらないため @Scheduled の設定に問題があるとばかり思ってそちらの調査に時間を割いてしまっていた。

## スレッドプールサイズの変更方法

上記のような状況を回避するにはスレッドのプールサイズを変更してやれば良い。
(もちろん ClassB の状況次第では延々とポーリングし続けて、いつまでたっても一回の処理が終了しない、というのも解決すべき問題であるが。。。)

その方法について後述の参考サイトに例が記載してあるが、それを転載すると

```java:ExampleConfiguration.java
@Bean
public ThreadPoolTaskScheduler  taskScheduler(){
    ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
    taskScheduler.setPoolSize(2); // プールサイズを変更する
    return  taskScheduler;
}
```

といった感じで ThreadPoolTaskScheduler を @Bean 登録してやり、その際に **setPoolSize でプールサイズを設定**してやれば良い。

## 参考

* [Does spring @Scheduled annotated methods runs on different threads?](https://stackoverflow.com/questions/21993464/does-spring-scheduled-annotated-methods-runs-on-different-threads)
* [27. Task Execution and Scheduling](https://docs.spring.io/spring/docs/4.0.x/spring-framework-reference/htmlsingle/#scheduling)

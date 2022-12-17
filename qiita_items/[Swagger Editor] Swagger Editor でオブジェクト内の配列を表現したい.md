[Swagger Editor](https://editor.swagger.io/#) を使う機会があったのだが、その際にオブジェクトの中に配列を持つデータ構造を表現するのにハマったので備忘録として残す。

## やりたいこと

次のような ｢ **オブジェクトの中に配列を持つ** ｣ データを Swagger Editor を使って表現したい｡

```json:表現したいデータ
{
  "data": {
    "id": 0000001,
    "name": "hogehogehogehoge",
    "child_id": [
      "child_00001",
      "child_00002",
      "child_00003",
      "child_00004"
    ]
  }
}
```

## 実現方法

Swagger Edtitor は YAML で記述するので次の内容となる。

### 単純にそのまま書く場合

```yaml:実現するための記法
definitions:
  SampleData:
      type: object
      properties:
        data:
          type: object
          properties:
            id:
              type: string
              example: 0000001
            name:
              type: string
              example: hogehogehogehoge
            child_id:
              type: array
              items:
                type: string
              example:
                - child_00001
                - child_00002
                - child_00003
                - child_00004
```

### $ref を使う場合

```yaml:実現するための記法($refあり)
definitions:
  SampleData:
      type: object
      properties:
        data:
          type: object
          properties:
            id:
              type: string
              example: 0000001
            name:
              type: string
              example: hogehogehogehoge
            child_id:
              $ref: "#/definitions/ChildIdList"

  ChildIdList:
    type: array
    items:
      type: string
      example:
        - child_00001
        - child_00002
        - child_00003
        - child_00004
```

## どこにハマっていたか

```child_id``` を配列で表現する記法がわからなかった。

### 試行錯誤の例1(エラーになる)

```yaml:試行錯誤の例1(エラーになる)
child_id:
  type: array
  items:
    - child_00001
    - child_00002
    - child_00003
    - child_00004
```

としたらエラーになり、

### 試行錯誤の例2_YAML(配列の中に配列ができる)

```yaml:試行錯誤の例2_YAML(配列の中に配列ができる)
child_id:
  type: array
  example:
    type: string
    items:
      - child_00001
      - child_00002
      - child_00003
      - child_00004
```

としたら

```json:試行錯誤の例2_JSON(配列の中に配列ができる)
"child_id": [
  [
    "child_00001",
    "child_00002",
    "child_00003",
    "child_00004"
  ]
],
```

となったり、とかなり時間をロスした。


## まとめにかえて

試行錯誤して上記を解決したあとに気づいたのだが、[Swagger Editor](https://editor.swagger.io/#) には **Convert to YAML** という機能があって、JSON データを YAML にコンバートしてくれる。
これを利用すればもう少し早く解決できたと思うので、今後記法でつまづいたら

1. まず JSON で実現したデータを書く
1. そのデータを Swagger Editor で YAML にコンバートする

という手順を行う、というところでまとめにかえる。

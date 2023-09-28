# かんたんに [Poetry](https://python-poetry.org/) と [Streamlit](https://docs.streamlit.io/) のご紹介

## [Poetry](https://python-poetry.org/)

Python にはパッケージ管理ツールとして [pip](https://pip.pypa.io/en/stable/) がありますが、Poetry はパッケージ管理だけではなく依存関係の解決や仮想環境の管理、ビルドやパッケージの公開まで行える Python のエコシステムの一つです。

## [Streamlit](https://docs.streamlit.io/)

Streamlit は Python の Web フレームワークの一つです。  
Python コードで Web ページを簡単に作れるのが特徴で、気軽に Web アプリの開発を楽しむことができます。  
そして Python コードで実装できるということで、 Python のライブラリを用いたデータ分析処理等を手軽に Web ページ上で可視化できる、といった利点があります。

# Web アプリ開発していきます

Poetry や Streamlit についてはそれぞれ単体で記事がいくつも書ける題材ですが本記事では割愛させて頂き、両者を用いた Web アプリの作成に焦点をあてて触れていきたいと思います。

## 環境

この記事の内容は次の環境で確認しております。

| 環境       | バージョン                         | 備考                         |
| --------- | --------------------------------- | --------------------------- |
| OS        | macOS Monterey v12.3              |                             |
| zsh       | zsh 5.8 (x86_64-apple-darwin21.0) | `zsh --version` で確認       |
| Python3   | Python 3.8.12                     | `python --version` で確認    |
| Poetry    | Poetry (version 1.2.2)            | `poetry --version` で確認    |
| Streamlit | Streamlit, version 1.12.2         | `streamlit --version` で確認 |

## Poetry のインストール

[公式の手順](https://python-poetry.org/docs/#installing-with-the-official-installer) にそって Poetry をインストールします。
次のコマンドを実行します。

```bash
% curl -sSL https://install.python-poetry.org | python3 -
```

インストールが成功すると、次の内容が出力されます。

```bash
Retrieving Poetry metadata

# Welcome to Poetry!

This will download and install the latest version of Poetry,
a dependency and package manager for Python.

It will add the `poetry` command to Poetry's bin directory, located at:

/Users/user_name/.local/bin

You can uninstall at any time by executing this script with the --uninstall option,
and these changes will be reverted.

Installing Poetry (1.2.2): Done

Poetry (1.2.2) is installed now. Great!

To get started you need Poetry's bin directory (/Users/user_name/.local/bin) in your `PATH`
environment variable.

Add `export PATH="/Users/user_name/.local/bin:$PATH"` to your shell configuration file.

Alternatively, you can call Poetry explicitly with `/Users/user_name/.local/bin/poetry`.

You can test that everything is set up by executing:

`poetry --version`
```

## PATH の設定と反映

```bash
# .zshrc を編集, 次の行を追加
export PATH="$HOME/.local/bin:$PATH"
```

```bash
# シェルを再起動して .zshrc を再読み込み
% exec $SHELL -l
```

## プロジェクトのセットアップ

<p>1. プロジェクトを作成する</p>

次のコマンドを実行し、プロジェクトを作成します。今回は `streamlit-work` というプロジェクト名にします。

```bash
% poetry new streamlit-work
Created package streamlit_work in streamlit-work
```

指定した名前でプロジェクトディレクトリが作成されたことを確認します。

```bash
% ls -l
drwxr-xr-x   6 user_name  staff      192 11 24 20:04 streamlit-work
```

プロジェクトディレクトリに移動し、構成を確認します。

```bash
% cd streamlit-work/
% tree -F
./
├── README.md
├── pyproject.toml
├── streamlit_work/
│   └── __init__.py
└── tests/
    └── __init__.py

2 directories, 4 files
```

<p>2. 仮想環境を作成する</p>

仮想環境を作成します。  
下記コマンドを実行することで、 pyproject.toml に登録された package がインストールされた仮想環境が作成されます。

```bash
% poetry install
# /path/to/caches/ はデフォルトで指定される仮想環境の path です
# デフォルト path については下記をご参照ください
# https://python-poetry.org/docs/configuration/#cache-directory
Creating virtualenv streamlit-work-WVbVqaqk-py3.10 in /path/to/caches/virtualenvs
Updating dependencies
Resolving dependencies... (0.1s)

Writing lock file

Installing the current project: streamlit-work (0.1.0)
```

次のコマンドで仮想環境が出来ていることを確認します。

```bash
# 仮想環境を確認する
# ( poetry install 実行時のログに出力された仮想環境が出力されます )
% poetry env list
streamlit-work-WVbVqaqk-py3.10 (Activated)
```

ここまでで Poetry のインストール〜プロジェクトのセットアップが完了しました。  
次は Streamlit のインストール〜Web アプリの作成を行っていきます。

## Streamlit のインストール

次のコマンドを実行し、プロジェクトに Streamlit を追加します。

```bash
% poetry add streamlit
Using version ^1.15.1 for streamlit

Updating dependencies
Resolving dependencies... (102.6s)

Writing lock file

Package operations: 46 installs, 0 updates, 0 removals

  • Installing six (1.16.0)
  • Installing attrs (22.1.0)
  • Installing markupsafe (2.1.1)
  • Installing numpy (1.23.5)
  • Installing pyrsistent (0.19.2)
  • Installing python-dateutil (2.8.2)
  • Installing pytz (2022.6)
  • Installing smmap (5.0.0)
  • Installing tzdata (2022.6)
  • Installing certifi (2022.9.24)
  • Installing charset-normalizer (2.1.1)
  • Installing commonmark (0.9.1)
  • Installing decorator (5.1.1)
  • Installing entrypoints (0.4)
  • Installing gitdb (4.0.10)
  • Installing idna (3.4)
  • Installing jinja2 (3.1.2)
  • Installing jsonschema (4.17.1)
  • Installing pandas (1.5.2)
  • Installing pygments (2.13.0)
  • Installing pyparsing (3.0.9)
  • Installing pytz-deprecation-shim (0.1.0.post0)
  • Installing toolz (0.12.0)
  • Installing urllib3 (1.26.13)
  • Installing zipp (3.10.0)
  • Installing altair (4.2.0)
  • Installing blinker (1.5)
  • Installing cachetools (5.2.0)
  • Installing click (8.1.3)
  • Installing gitpython (3.1.29)
  • Installing importlib-metadata (5.0.0)
  • Installing packaging (21.3)
  • Installing pillow (9.3.0)
  • Installing protobuf (3.20.3)
  • Installing pyarrow (10.0.1)
  • Installing pydeck (0.8.0)
  • Installing pympler (1.0.1)
  • Installing requests (2.28.1)
  • Installing rich (12.6.0)
  • Installing semver (2.13.0)
  • Installing toml (0.10.2)
  • Installing tornado (6.2)
  • Installing typing-extensions (4.4.0)
  • Installing tzlocal (4.2)
  • Installing validators (0.20.0)
  • Installing streamlit (1.15.1)
```

このような感じで依存関係含めてインストールしてくれます。

## 最初のページを作成

ここからは Web アプリを作成していきます。

`index.py` として最初のページを作成します。  
次のコードを `プロジェクトルートディレクトリ/streamlit_work/index.py` として保存してください。  
コードの説明はコメントをご参照ください。

```python:index.py
# streamlit を使用するために import します
import streamlit as st

# ページタイトルを表示します
st.title('Streamlit app')

# `h2` タグとして表示するために `markdown` 記法を使用しています
st.markdown('## Streamlit の紹介')
st.markdown('Python のフレームワーク, Streamlit の紹介です. このページは Streamlit を使って実装してます.')

# こちらも markdown 記法を利用して `h3` タグと `a` タグを実現してます
st.markdown('### API Reference')
link = '[Streamlit API](https://docs.streamlit.io/library/api-reference)'
comment = f'{link}: Streamlit の API については左こちらのリンクを参照してください.'

# unsafe_allow_html=True とすることで html タグを html として解釈させます( 公式では非推奨としています )
# 下記リファレンスをご参照ください
# https://docs.streamlit.io/1.12.0/library/api-reference/text/st.markdown#stmarkdown
st.markdown(comment, unsafe_allow_html=True)
```

保存したら次のコマンドを実行してください。

```bash
% poetry run streamlit run streamlit_work/index.py

  You can now view your Streamlit app in your browser.

  Local URL: http://localhost:8501
  Network URL: http://192.168.2.186:8501

  For better performance, install the Watchdog module:

  $ xcode-select --install
  $ pip install watchdog
```

自動でブラウザが起動し、上記で保存したページが確認できると思います。

<img width="1307" alt="index" src="https://user-images.githubusercontent.com/85983748/204239332-f494fdb8-849b-4b35-b070-48fcd96e97f8.png">

## マルチページの作成

streamlit ではマルチページにも対応できます。([公式-Adding pages](https://docs.streamlit.io/library/get-started/multipage-apps#adding-pages))  
プロジェクトディレクトリ直下の `streamlit_work/` 配下に `pages/` ディレクトリを作成してください。

```bash
% mkdir -p streamlit_work/pages
```

ここでは入力フォームを実装してみます。上記で作成した `streamlit_work/pages/` 配下に次のコードを `forms.py` として保存してください。  
これだけでマルチページが実現されます。

```python:forms.py
import streamlit as st
import datetime

# st.form
st.markdown('### 入力ウィジェットを使った form のサンプル')
link_form = '[form](https://docs.streamlit.io/library/api-reference/control-flow)'
link_input_widjets = '[input widjets](https://docs.streamlit.io/library/api-reference/widgets#input-widgets)'
st.markdown(f'これは {link_input_widjets} を使った {link_form} のサンプルです. ')
st.markdown(f'入力ウィジェットについては {link_input_widjets} を参照してください.')

# ページレイアウトを 2分割 します
col1, col2 = st.columns(2)

# ページ左部の実装, 入力フォームを作っていきます
with col1:
    st.caption('入力フォームのサンプル')
    with st.form(key='profile_form'):
        name = st.text_input('名前')
        address = st.text_input('住所')

        # ラジオボタンの実装, select ボックスにしたい場合は st.select とすれば良いです
        age_category = st.radio('年齢層', ('18歳未満', '18歳以上'))

        # マルチセレクタの実装, 第1引数にはラベル, 第2引数には選択肢を指定します
        # Web ページ上で選択した項目が入力欄に反映されていきます
        hobby = st.multiselect(
            '趣味',
            ('スポーツ', '読書', 'プログラミング', '映画鑑賞', '釣り', '料理')
        )

        # submit 処理を擬似的に実装
        # submit_btn(送信) をクリックすることで入力フォームで入力した情報が表示されます
        submit_btn = st.form_submit_button('送信')
        cancel_btn = st.form_submit_button('キャンセル')

        if submit_btn:
            st.text(f'ようこそ {name} さん. {address} に書籍を送りました.')
            st.text(f'年齢層: {age_category}')
            st.text(f'趣味: {", ".join(hobby)}')

# ページ右部の実装, 色々なウィジェットを試します
with col2:
    st.caption('ウィジェットいろいろ')

    # チェックボックス,　スライダー, デートピッカー, カラーピッカー が簡単に実現できます。
    check_box = st.checkbox('チェックボックス')
    slider = st.slider('長さ', min_value=0, max_value=100)
    datepicker = st.date_input(
        '開始日',
        datetime.date(2022, 9, 1)
    )
    colorpicker = st.color_picker('テーマカラー', '#00f293')

```

再度ブラウザを見ていただくと、ページ左部にペインが現れ `index` と `forms` といったページリンクが表示されていると思います。  
( Web アプリを停止してしまっている場合は、再度起動して確認してください )

<img width="1311" alt="forms" src="https://user-images.githubusercontent.com/85983748/204239379-9ad559bc-1193-4f67-93d6-de5ada3b5d0c.png">

## データ操作を試す

最後に csv ファイルから温度データを読み込んでグラフ表示する例をお見せしたいと思います。  
プロジェクトディレクトリ直下の `streamlit_work/` 配下に `data/` ディレクトリを作成してください。

```bash
% mkdir -p data
```

上記で作成した `streamlit_work/data/` 配下に csv データを `temperature.csv` として保存してください。

```csv:temperature.csv
月,2021年,2022年
1,5.4,7.1
2,8.5,8.3
3,7.2,8.0
4,10.4,11.2
5,22.5,20.0
6,20.1,24.4
7,30.2,34.7
8,35.5,36.2
9,33.2,32.5
10,28.3,25.9
11,18.5,16.1
12,13.6,10.8
```

次に `streamlit_work/pages/` ディレクトリ配下に次のコードを `temperature.py` として保存してください。

```python:temperature.py
import streamlit as st
import pandas as pd

# データ操作
st.markdown('### データ操作 のサンプル')
link_chart_widjets = '[chart widjets](https://docs.streamlit.io/library/api-reference/charts#chart-elements)'
st.markdown(f'これは {link_chart_widjets} を使ったサンプルです. ')
st.markdown(f'チャートウィジェットについては {link_chart_widjets} を参照してください.')

# csv データを読み込んでオブジェクトを格納します
# あとは Streamlit の API 経由でこのオブジェクトを扱うだけで色々なチャートでデータを見ることができます
data_file = pd.read_csv('./streamlit_work/data/temperature.csv', index_col='月')

# レイアウト調整, ページレイアウトを 4分割 します
# ここでは温度データをテーブル、折れ線グラフ、棒グラフ(2021年), 棒グラフ(2022年) で表示させてみます
col1, col2, col3, col4 = st.columns(4)
with col1:
    st.caption('温度データをテーブルで表示')
    st.table(data_file)

with col2:
    st.caption('温度データを折れ線グラフで表示')
    st.line_chart(data_file)

with col3:
    st.caption('温度データを棒グラフで表示(2021年)')
    st.bar_chart(data_file['2021年'])

with col4:
    st.caption('温度データを棒グラフで表示(2022年)')
    st.bar_chart(data_file['2022年'])

```

ブラウザを見ていただくと、ページ左部のペインに `index` と `forms` に加えて `temperature` といったページリンクが表示されていると思います。  
( Web アプリを停止してしまっている場合は、再度起動して確認してください )

各チャートは最大化したり画像保存したりといった操作も行えます。  
是非ご自身で触れてみてください。

<img width="1310" alt="temperature" src="https://user-images.githubusercontent.com/85983748/204239475-1db7618d-0161-4bb3-9134-2a33725c2457.png">

# アプリの構成と起動

最後に本記事で作成した Web アプリの構成と起動について振り返りたいと思います。  
index.py の項目でアプリ起動コマンドについて触れましたが、改めてこちらでも扱っておきます。

## アプリの構成

```bash
% tree -F
./
├── README.md
├── poetry.lock              # パッケージ管理ファイル(ロックファイル)
├── pyproject.toml           # パッケージ管理ファイル
├── streamlit_work/          # streamlit Web アプリルートディレクトリ
│   ├── __init__.py
│   ├── data/
│   │   └── temperature.csv  # 温度データ
│   ├── index.py             # トップページ
│   └── pages/               # マルチページを配置していく
│       ├── forms.py         # 入力フォームサンプルページ
│       └── temperature.py   # 温度データ可視化のサンプルページ
└── tests/
    └── __init__.py

4 directories, 9 files
```

## アプリの起動

poetry 経由で streamlit の Web アプリを起動するには次のコマンドを実行します。

```bash
# poetry で streamlit を実行し、streamlit は Web アプリのトップページを指定して起動する
% poetry run streamlit run streamlit_work/index.py
```

冒頭触れた Poetry を使ってのビルドやパッケージ公開につきましては、ご興味ある方は[公式ページ -> Publishing to PyPI](https://python-poetry.org/docs/libraries#publishing-to-pypi)をご参照頂ければと思います。

# まとめにかえて

いかがでしたでしょうか。Poetry + Streamlit の組み合わせで簡単に Web アプリが作成できることがお分かり頂けたかと思います。  
Streamlit では [html を静的に埋め込む API](https://docs.streamlit.io/library/components/components-api) も用意されているので、html の構成を任意に組むことも可能です。

また [Streamlit Cloud](https://streamlit.io/cloud)というサービスを利用することで、クラウド上に Web アプリを気軽にデプロイ & 公開する手段も用意されています。ご興味ある方はぜひお試しください。

# 参考ドキュメント

- Poetry
  - [Poetry](https://python-poetry.org/)
  - [Poetry documentation](https://python-poetry.org/docs/)
- Streamlit
  - [Streamlit](https://streamlit.io/)
  - [Streamlit documentation](https://docs.streamlit.io/)


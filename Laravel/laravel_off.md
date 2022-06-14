# もくじ

[toc]

# 1）Laravel 共通編

・ファイルは全て「UTF-8」であること

・レガシー SQL 文 の廃止。 必ず Eloquent 化すること（＝可読性の向上）
　 → Laravel 移植の際に、現行 SQL はコメントとして残しておいてください（＝移植の妥当性＆品質チェックのため）
　 → tail -f storage/logs/laravel-2022-04-14.log に出力される SQL 文とあっているかをチェックしてください。
　 → Eloquent については、 [[ドキュメント/Laravel/01.Eloquent(Model)の使い方]] を参照ください

・MVC の分離。 ロジックの中の HTML を極力 blade へ移植すること（＝可読性の向上）

・レガシーメール送信機能の廃止。 Laravel 標準のメール送信機能で実装しなおすこと！（メール品質の向上）

・init2.inc 内にある Pure PHP ベースの関数は、helper 関数にすることで、移植コストの軽減を図る  
 → 原則、１件フェッチの DecCm 系は廃止したい・・・  
　 → 極力、親となるモデルに対するリレーション（belongsto）を定義してください・・  
 → リレーション定義については、 [[ドキュメント/Laravel/02.リレーション定義]] を参照ください

→ 複雑な SQL の場合は、getXXXXAttribute で定義してください。

・Model は、原則 AtlinkBaseModel を継承すること（勝手にモデルを作らない・・・）  
・同一モデルに対する検索条件を各ビジネスロジックで、何度も書かない！
　（特に、複合キーの場合は、必要なパラメータをモデル内にスコープ定義すること！）

・論理削除は、AtlinkSoftDelete を使うことで品質を統一する  
・各 Eloquent での client カラム指定は廃止（＝可読性の向上）  
　 → 一応、client カラムが定義されていなくても動くようにフレームワーク側で考慮済
　 → 暗黙的／明示的に、2 重定義されてても、SQL 的には問題ないですが、明示的には不要にしたい。
　 （各自で、laravel.log を参照し、SQL 文をチェックしてください）

・医療機関データや ENV は、共通部品として以下のように取得する

```
$clnt = app('clntCollection');
$envs = app('envCollection');
```

・blade 内であれば、直接 $clnt および、$envs を参照することも可能 （フレームワーク側で変数定義していますので、コントローラーから渡す必要はなし）

・また、コントローラーや blade 内で、ENV データを参照する場合は、ヘルパー「\_env」を利用する。

```
例： リライトカードを制御する変数「$RW_CARD」を参照する場合は、Laravel では以下のように定義する
_env('RW_CARD')
```

・日付計算を、SQL で行うのは原則廃止！！（パフォーマンス改善）

・リレーションを使った場合は、Eloquent モデルの with 指定を行うこと！！（ループ処理におけるパフォーマンス劣化を防ぐため）
　 → laravel.log や debugbar を使って、SQL 発行回数をチェックしてください・・・

# 2）医事画面編

・ステップ 1 から、メニュー画面を Laravel 化するので、旧ファイルのメンテナンスは不要とする方針（要すり合わせ）

以下、Laravel 化する際に考慮すべきこと
　 → ENV での表示制御
　 → if クライアントでの表示制御
　 → ブラウザでの表示制御

・各画面の css / js ファイルの撤廃！！

各 blade の ＠section("css") or @section("js") 内に定義することで、
　 if クライアント対応等しやすくする！！！（ご参考： home.blade.php 参照）
　 → コード量が多いときは、home.js.blade.php 等でファイル分割＆import すること！！

# ３）バッチ編

・原則 AtlinkBaseCommnad を継承することで、各バッチにおいては、ビジネスロジックの実装に専念する
・Pstl の制御は原則、AtlinkBaseCommand に任せる！！
・pgname を定義し、$this->info メソッドを使うことで、Laravel バッチからのログ制御も統一する

# ４）ネーミングルール

一般的な Laravel 　特有のコーディング規約は、以下をご参照ください。
https://www.laravuejs.dev/tutorials/laravel-s1/laravel-115/#laravel%E7%89%B9%E6%9C%89%E3%81%AE%E3%82%B3%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E8%A6%8F%E7%B4%84

なんとなく記載されているとおもいますが、タブや空白の位置なども可読性を統一させるためには、ルールが決まっています。
PHP コーディング規約は、以下をご参照ください。
https://qiita.com/a24dai/items/25e69eac8676e3751510

※ アットリンク独自ネーミングルールは、別途作成中（近日中）

## URL マッピング

### アットリンクの場合、 /routes/atlink/web.php をルートとする！！

例：Web 予約サイトを Laravel 化した場合は、 /routes/yoyaku/web.php にする予定

### 機能ごとに、以下のようにマッピング定義ファイルを分離する

例：/routes/atlink/resv.php

| 機能名         | route ファイル名 | 備考                                       |
| -------------- | ---------------- | ------------------------------------------ |
| 共通           | web.php          | ログイン画面、ヘッダーなどの共通画面を定義 |
| 予約受付       | (仮)resv.php     |                                            |
| 業務支援       | xxxxx.php        |                                            |
| マーケティング | xxxx.php         |                                            |
| システム管理   | master.php       |                                            |

### ルート名については必ず設定すること。2 重定義は禁止。

ルート名の構成：atlink.[機能].[名称].[画面]
システム管理のユーザーメンテナンスの場合は「atlink.master.user.index」となる。
一覧、登録、更新といった一般的な画面は以下のルート名を利用すること（参考: laravel\routes\atlink\master.php）

| 画面     | ルート名 | メソッド |
| -------- | -------- | -------- |
| 一覧画面 | index    | GET      |
| 登録画面 | create   | GET      |
| 登録処理 | store    | POST     |
| 詳細画面 | show     | GET      |
| 編集処理 | edit     | GET      |
| 更新処理 | update   | PATCH    |
| 削除処理 | destory  | DELETE   |

## フォルダ名

上記の URL マッピングに紐づくようなファイルの管理単位が一目でわかるようなネーミングにすること！！

## モデル

・原則、/app\Models\*\*\*\*.php に配置する
・原則、app/Models/AtlinkBaseModel.php クラスを継承させること！！
・各モデルで、コンストラクタをオーバーライドする場合は、以下のように実装してください。

```
    public function __construct($attributes = array()) {

        // 親クラスの初期化をコール
        parent::__construct($attributes);

        // 子クラス独自の実装

    }
```

## コントローラー

原則、/app/Controllers/Atlink/\*\*\*\*.php に配置する

## ブレード

原則、/resources/views/atlink/機能名/\*\*\*\*blade.php に配置する

### アットリンク各管理画面の blade では、以下のマスタ blade を include すること！！！

```
resources/views/layouts/atlink/clnt.blade.php
```

### ポップアップ画面は、以下のマスタ blade を include すること！！

```
resources/views/layouts/atlink/clnt_pop.blade.php
### 共通部品は、/resources/views/atlink/partials/****blade.php  に定義する

### ツールチップ（i マーク）の呼び出し方は以下の通り

```

{!! uiInformation(’ログインユーザ’ID 'カテゴリ（※）', ’表示したい’タイトル) !!}

```

※カテゴリはcont/data/info配下にあるCSVファイル名の「info_XXXXX」のXXXXX部分を指す。
　これまで使用するページで各々のファイルを読み込んでいたが、
　アットリンク Laraval では helpler 関数として共通化を実施。

## カスタマイズ
### 原則、クライアント特有のプログラムは以下のフォルダに置くこと！！
例： customize/クライアントID/ ****

## バッチ
### Laravel バッチの仕様上、以下のようなネーミングとする （バッチは、pstl の制御等があるので、現行踏襲仕方なし・・・）
例： batch:clntb001

### カスタマイズしたい場合は、以下のようなネーミングとする
例：クライアントID:clntb001
※  customize/クライアントID/Console/Commands/Batch/clntb001.php をフレームワークで参照します！！

※ 機能名や処理が見てわかるものが理想！！
※ ただし、管理画面系は、プログラムID を廃止したい・・・


# 5）環境定義編
・クライアントIDは、 /home/atlink/lib/env.inc  から取得する。

・laravel の  .env は、クライアント依存はさせない。
　⇒ カスタマイズ定義したい場合は、/customize/クライアントID/.env  に定義する。

# 6) ENUM化

「deccm.decid」や「mcdt.mcat」など
マジックナンバーで管理されている定数の一覧をENUM化する。

詳細は [Enum定義](https://offshore-i.backlog.com/alias/wiki/1842859) の詳細ページを確認
```

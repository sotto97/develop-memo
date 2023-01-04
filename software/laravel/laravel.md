# Laravel

## よく使うコマンド

```bash
$ php artisan make:model Report -m -c -r
$ php artisan make:controller ReportController --resource

# マイグレーションファイルも同時生成
$ php artisan make:model Models/Report -m

# Postsというテーブルにbodyを追加したい場合
$ php artisan make:migration add_body_to_posts_table --table=posts
```

## 手順

- エラーメッセージの日本語化

```bash
php -r "copy('https://readouble.com/laravel/8.x/ja/install-ja-lang-files.php', 'install-ja-lang.php');"
php -f install-ja-lang.php
php -r "unlink('install-ja-lang.php');"
```

```php
// config/app.php
'locale' => 'ja',
```

- 論理削除カラムを作成

```bash
# Productsというテーブルにdeleted_atを追加する
$ php artisan make:migration add_softDelete_to_products_table --table=products
```

```php
// migrationファイル内
public function up()
{
    Schema::table('products', function (Blueprint $table) {
        $table->softDeletes();
    });
}

/**
 * Reverse the migrations.
 *
 * @return void
 */
public function down()
{
    Schema::table('products', function (Blueprint $table) {
        $table->dropSoftDeletes();
    });
}
```

- ログイン機能

```bash
composer require laravel/ui:^3

# LOGIN機能
php artisan ui vue --auth
npm install && npm run dev

# テーブル作成
php artisan migrate
```

- TailwindCSS

```bash
npm install
npm install -D tailwindcss
npx tailwindcss init
```

```bash
# tailwind.config.js
content: [
    './storage/framework/views/*.php',
    './resources/**/*.blade.php',
    './resources/**/*.js',
    './resources/**/*.vue',
],
```

```bash
# resources/css/app.css ←ここが前と比較して変わった。前はapp.scssに記載していた
@tailwind base;
@tailwind components;
@tailwind utilities;
```

```bash
# webpack.mix.js
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css', [
        require('tailwindcss'),
    ]);
```

```bash
# layouts/app.blade.php
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link href="/css/app.css" rel="stylesheet">
</head>
```

- Vue

```bash
# パッケージのインストール
composer require laravel/ui

# ログイン／ユーザー登録スカフォールドを生成
php artisan ui vue --auth
npm install
npm run dev
```

- Vue Router

```bash
npm install vue-router
```

```js
// app.js
require("./bootstrap");

import Vue from "vue";
import VueRouter from "vue-router";

window.Vue = Vue;
Vue.use(VueRouter);

const app = new Vue({
	el: "#app",
});
```

- Vuetify を使用するには

```bash
npm i vuetify
```

```js
// resources/js/app.js
import Vuetify from 'vuetify';
Vue.use(Vuetify);

// ...中略

const app = new Vue({
  el: ‘#app’,
  vuetify: new Vuetify(),
});
```

app.scss に以下を追加

```scss
// vuetify
@import "~vuetify/dist/vuetify.min.css";
@import "~@mdi/font/css/materialdesignicons.min.css";
```

ソースの変更を即座に同期して反映してくれる browserSync

```
// webpack.mix.js
mix.js("resources/js/app.js", "public/js").version()
	.vue()
	.sass("resources/sass/app.scss", "public/css")
	.postCss("resources/css/app.css", "public/css", [require("tailwindcss")])
	// ↓ここを追加
	.browserSync({
		files: ["resources/**/*", "config/**/*", "routes/**/*", "app/**/*", "public/**/*"],
		proxy: {
			target: "http://localhost:8000",
		},
		open: false,
		reloadOnRestart: true,
	});
```

## 規約

### ルーティング

Vue の route に関しては、router.js に記載
Laravel の route に関しては、api.php に記載

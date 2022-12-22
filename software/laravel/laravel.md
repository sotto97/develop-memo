# Laravel

## 手順

- TailwindCSS

```
npm install

npm install -D tailwindcss

npx tailwindcss init
```

```
// tailwind.config.js
content: [
    './storage/framework/views/*.php',
    './resources/**/*.blade.php',
    './resources/**/*.js',
    './resources/**/*.vue',
],
```

```
// resources/css/app.css ←ここが前と比較して変わった。前はapp.scssに記載していた
@tailwind base;
@tailwind components;
@tailwind utilities;
```

```
// webpack.mix.js
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css', [
        require('tailwindcss'),
    ]);
```

```
// layouts/app.blade.php
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link href="/css/app.css" rel="stylesheet">
</head>
```

- Vue

```

// パッケージのインストール
composer require laravel/ui

// ログイン／ユーザー登録スカフォールドを生成
php artisan ui vue --auth
npm install
npm run dev

```

- Vue Router

```

npm install vue-router

```

```app.js
require('./bootstrap');

import Vue from 'vue';
import VueRouter from 'vue-router';

window.Vue = Vue;
Vue.use(VueRouter);

const app = new Vue({
    el: '#app',
});
```

- Vuetify

```
npm i vuetify
```

```resources/js/app.js
import Vuetify from 'vuetify';
Vue.use(Vuetify);

// ...中略

const app = new Vue({
  el: ‘#app’,
  vuetify: new Vuetify(),
});
```

app.scss に以下を追加

```
// vuetify
@import "~vuetify/dist/vuetify.min.css";
@import "~@mdi/font/css/materialdesignicons.min.css";
```

ソースの変更を即座に同期して反映してくれる browserSync

```webpack.mix.js
mix.js("resources/js/app.js", "public/js")
    .version()
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
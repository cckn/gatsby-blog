---
title: 웹팩 플러그인 사용법 및 주요 플러그인
date: "2021-01-03T01:44:03.284Z"
description: "Webpack에서 자주 쓰이는 플러그인들에 대해 알아보자"
---

> - 본 글은 김정환님의 강의 [프론트엔드 개발환경의 이해와 실습](https://www.inflearn.com/course/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD/dashboard)를 수강 후 정리 및 복습하고자 작성한 포스트입니다.
> - 강의 내용과 제가 이해한 방향이 다소 다를 수 있습니다.

## Webpack Plugin?

각각의 파일을 해석하는 데 들어가는 로더와 달리 번들된 경과물을 처리하는 것이 플러그인이다.

5개의 파일이 있다고 하면 로더는 각각의 파일을 모듈화 할 때 호출되고(5번), 플러그인은 최종 1번 호출된다.

## 자주 쓰이는 플러그인

> - 해당 소스코드들은 웹팩 공통부분을 들어내고 플러그인 부분만 남겨놓은 `webpack.config.js` 파일들입니다.
> - 전체 소스코드는 최하단에 배치하였습니다.

### Banner Plugin

번들링된 파일의 상단에 배너(텍스트)를 달아주는 플러그인이다.

webpack 기본 플러그인이기에 따로 설치는 필요없다.

빌드시의 데이터들을 자동으로 기록해주는 방식으로 사용하면 유용할 듯 하다.

```js
// webpack.config.js

const webpack = require("webpack")
const childProcess = require("child_process")

const removeNewLine = buffer => {
  return buffer.toString().replace("\n", "")
}
module.exports = {
  plugins: [
    new webpack.BannerPlugin({
      banner: `
        Build Date :: ${new Date().toLocaleString()}
        Commit Version :: ${removeNewLine(
          childProcess.execSync("git rev-parse --short HEAD")
        )}
        Auth.name :: ${removeNewLine(
          childProcess.execSync("git config user.name")
        )}
        Auth.email :: ${removeNewLine(
          childProcess.execSync("git config user.email")
        )}
  `,
    }),
  ],
}
```

`removeNewLine`는 childProcess의 결과물의 `\n`을 하나씩 삭제해준다.

위와 같이 사용하면 아래와 같이 `Build Date`, `Commit Version`, `user.name`, `user.email` 등을 빌드시마다 자동으로 기록할 수 있다.

```js
/*!
 *
 *         Build Date :: 2021. 1. 2. 오후 10:54:53
 *         Commit Version :: d131165
 *         Auth.name :: cckn
 *         Auth.email :: cckn.dev@gmail.com
 *
 */
```

### Define Plugin

각종 환경 변수들을 빌드시에 지정할 수 있는 플러그인이다.

개발 환경에 따라 서버 URL 등의 정보가 바뀌는 경우가 많은데 그럴 때 미리 지정해두면 실수를 방지할 수 있다.

```js
// webpack.config.js

const webpack = require("webpack")

module.exports = {
  plugins: [
    new webpack.DefinePlugin({
      TWO: JSON.stringify("1+1"),
      "api.domain": JSON.stringify("https://www.naver.com"),
    }),
  ],
}
```

### HTML Template Plugin

HTML을 동적으로 생성할 수 있는 플러그인이다.

해당 플러그인을 사용하면 `index.html`에서 `js file`을 로드하는 부분을 삭제해도 된다.

사용할 `html` 파일을 `template`로 지정 후에 사용하면 빌드 결과물이 생성되는 output 폴더에 index.html 파일이 생성된다.

`templateParameters`, `minify` 등의 옵션으로 생성될 `html` 파일에 원하는 처리를 해줄 수 있다.

아래 소스코드는 `NODE_ENV === 'development'`이면 `title`에 `"(개발용)"`이라는 접미사를 붙여주고

production인 경우 `minify`(공백, 줄바꿈, 주석 삭제)를 수행한다.

```bash
# install
npm i -D html-webpack-plugin
```

```js
// webpack.config.js

const HtmlWebpackPlugin = require("html-webpack-plugin")

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html",
      templateParameters: {
        env: process.env.NODE_ENV === "development" ? "(개발용)" : "",
      },
      minify:
        process.env.NODE_ENV === "production"
          ? { collapseWhitespace: true, removeComments: true }
          : false,
    }),
}
```

> 아래 파일의 `<title>Document <%= env %></title>` 부분은 [ejs](https://ejs.co/#promo)를 참조하자

```html
<!-- src/index.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document <%= env %></title>
  </head>
  <body>
    <div id="app"></div>
    <!-- 주석입니다.  -->
  </body>
</html>
```

Result

```html
<!-- dist/index.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>Document</title>
    <link href="main.css" rel="stylesheet" />
  </head>
  <body>
    <div id="app"></div>
    <script src="main.js"></script>
  </body>
</html>
```

자동으로 css와 main.js가 링크됐다.

### Clean Webpack Plugin

빌드되고 사용되지 않는 찌꺼기 파일들이나 캐시처럼 남아있는 파일들을 빌드 전에 삭제해주는 플러그인이다.

사용법은 제일 간단하다.

```bash
# install
npm i -D clean-webpack-plugin
```

```js
// webpack.config.js

const { CleanWebpackPlugin } = require("clean-webpack-plugin")
module.exports = {
  plugins: [
    new CleanWebpackPlugin(),
}
```

### Mini Css Extract Plugin

webpack 기본 설정은 css파일이 js파일에 같이 번들링 된다.

js 파일이 비대해질 경우 그만큼 페이지 로딩 시간이 길어진다.

이럴 때에는 js파일과 css 파일을 분리하는 편이 도움이 된다.

```bash
# install
npm i -D mini-css-extract-plugin
```

```js
// webpack.config.js

const MiniCssExtractPlugin = require("mini-css-extract-plugin")

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          process.env.NODE_ENV === "production"
            ? MiniCssExtractPlugin.loader
            : "style-loader",
          "css-loader",
        ],
      },
    ],
  },
  plugins: [
    ...(process.env.NODE_ENV === "production"
      ? [new MiniCssExtractPlugin({ filename: "[name].css" })]
      : []),
  ],
}
```

Mini Css Extract Plugin은 플러그인 뿐 아니라 loader도 손을 봐줘야 한다.

## 전체 소스코드

```js
// webpack.config.js

const path = require("path")
const webpack = require("webpack")
const childProcess = require("child_process")
const HtmlWebpackPlugin = require("html-webpack-plugin")
const { CleanWebpackPlugin } = require("clean-webpack-plugin")
const MiniCssExtractPlugin = require("mini-css-extract-plugin")

const env = process.env.NODE_ENV

const removeNewLine = buffer => {
  return buffer.toString().replace("\n", "")
}

module.exports = {
  mode: "development",
  entry: {
    main: "./src/app.js",
  },

  output: {
    path: path.resolve("./dist"),
    filename: "[name].js",
  },

  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          env === "production" ? MiniCssExtractPlugin.loader : "style-loader",
          "css-loader",
        ],
      },
      {
        test: /\.(png|jpg|gif|svg)$/,
        loader: "url-loader",
        options: {
          publicPath: "/dist/",
          name: "[name].[ext]?[hash]",
          limit: 20000, //2kb
        },
      },
    ],
  },

  plugins: [
    new webpack.BannerPlugin({
      banner: `
        Build Date :: ${new Date().toLocaleString()}
        Commit Version :: ${removeNewLine(
          childProcess.execSync("git rev-parse --short HEAD")
        )}
        Auth.name :: ${removeNewLine(
          childProcess.execSync("git config user.name")
        )}
        Auth.email :: ${removeNewLine(
          childProcess.execSync("git config user.email")
        )}
  `,
    }),
    new webpack.DefinePlugin({
      TWO: JSON.stringify("1+1"),
      "api.domain": JSON.stringify("https://www.naver.com"),
    }),
    new HtmlWebpackPlugin({
      template: "./src/index.html",
      templateParameters: {
        env: env === "development" ? "(개발용)" : "프로덕션",
      },
      minify:
        env === "production"
          ? { collapseWhitespace: true, removeComments: true }
          : false,
    }),
    new CleanWebpackPlugin(),

    ...(env === "production"
      ? [new MiniCssExtractPlugin({ filename: "[name].css" })]
      : []),
  ],
}
```

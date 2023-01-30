# Astro 無料で爆速サイトを作成する

YouTube で公開している Astro のチュートリアル動画のソースコードです。

## Astro の初期化とサイトの公開編

- [x] Astro のプロジェクトを作成する
- [x] GitHub にリポジトリを作成する
- [x] GitHub にソースコードをプッシュする
- [x] Netlify にサイトを公開する

[![【#1 Astro】爆速でWebサイトを無料で公開して稼ぐ方法 #astro](https://i9.ytimg.com/vi_webp/c1ohevLaas0/mqdefault.webp?v=63d6eeed&sqp=COyD3Z4G&rs=AOn4CLD5ni-pmnUe4Z0T1Ki3_TpkQemPRQ)](https://www.youtube.com/watch?v=c1ohevLaas0)

## Astro でブログサイトを作っていく編

- [x] Astro で Home, About, Blog のページを作成する
- [x] Layout を作成する
- [x] ブログ記事を作成する

[![【#2 Astro】爆速でWebサイトを無料で公開して稼ぐ方法 #astro](https://i9.ytimg.com/vi/_KBd8H19s0c/mqdefault.jpg?v=63d6efb6&sqp=COyD3Z4G&rs=AOn4CLC6y4HirjYGABSZGXp_YHIHoBwNQA)](https://www.youtube.com/watch?v=_KBd8H19s0c)

## Astro で広告を貼ってみる+Google AdSense で収益化する編

- [x] Astro で広告を貼る
- [x] Google AdSense で収益化する

[![【#1 Astro】爆速でWebサイトを無料で公開して稼ぐ方法 #astro](https://i9.ytimg.com/vi/FncGm2TKKCY/mqdefault.jpg?v=63d6f01f&sqp=COyD3Z4G&rs=AOn4CLCW23OadJJW0auyyffQuTaex-YXOw)](https://www.youtube.com/watch?v=FncGm2TKKCY)

## 概要

Astro, GitHub, Netlify を利用して無料でサイトを公開し、広告を貼るまで

作るサイト
https://tokyonight.net/

チュートリアルサイト
https://docs.astro.build/ja/tutorial/0-introduction/

GitHub
https://github.com/ekusiadadus/tokyonight-net

# Astro Starter Kit: Basics

```
npm create astro@latest -- --template basics
```

[![Open in StackBlitz](https://developer.stackblitz.com/img/open_in_stackblitz.svg)](https://stackblitz.com/github/withastro/astro/tree/latest/examples/basics)
[![Open with CodeSandbox](https://assets.codesandbox.io/github/button-edit-lime.svg)](https://codesandbox.io/s/github/withastro/astro/tree/latest/examples/basics)

> 🧑‍🚀 **Seasoned astronaut?** Delete this file. Have fun!

![basics](https://user-images.githubusercontent.com/4677417/186188965-73453154-fdec-4d6b-9c34-cb35c248ae5b.png)

## 🚀 Project Structure

Inside of your Astro project, you'll see the following folders and files:

```
/
├── public/
│   └── favicon.svg
├── src/
│   ├── components/
│   │   └── Card.astro
│   ├── layouts/
│   │   └── Layout.astro
│   └── pages/
│       └── index.astro
└── package.json
```

Astro looks for `.astro` or `.md` files in the `src/pages/` directory. Each page is exposed as a route based on its file name.

There's nothing special about `src/components/`, but that's where we like to put any Astro/React/Vue/Svelte/Preact components.

Any static assets, like images, can be placed in the `public/` directory.

## 🧞 Commands

All commands are run from the root of the project, from a terminal:

| Command                | Action                                           |
| :--------------------- | :----------------------------------------------- |
| `npm install`          | Installs dependencies                            |
| `npm run dev`          | Starts local dev server at `localhost:3000`      |
| `npm run build`        | Build your production site to `./dist/`          |
| `npm run preview`      | Preview your build locally, before deploying     |
| `npm run astro ...`    | Run CLI commands like `astro add`, `astro check` |
| `npm run astro --help` | Get help using the Astro CLI                     |

## 👀 Want to learn more?

Feel free to check [our documentation](https://docs.astro.build) or jump into our [Discord server](https://astro.build/chat).

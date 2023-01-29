---
layout: ../../layouts/MarkdownPostLayout.astro
title: "GitHub Actions で、Cron 定期実行でプルリクエストを作成してマージする"
pubDate: 2022-12-03
description: "GitHub Actions で、Cron 定期実行でプルリクエストを作成してマージする"
author: "@ekusiadadus"
image:
  url: "https://avatars.githubusercontent.com/u/70436490?s=400&u=a714da7802c65046265c6848887eecddfc58b5c0&v=4"
  alt: "GitHub Actions で、Cron 定期実行でプルリクエストを作成してマージする"
tags: ["GitHub Actions", "Cron", "GitHub CLI"]
---

# GitHub Actions で、自動でプルリクを作成してマージする方法

`dev` -> `stage` へのプルリクエストの作成とマージをする機会があり、定期的に実行できたら楽なので GitHub Actions で作成してみました。

```yml
name: Auto merge dev2stage

on:
  schedule:
    - cron: "0 15 * * *"

jobs:
  auto-merge-dev2stage:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
        with:
          ref: dev
      - name: 【定期実行】Create Pull Request dev2stage
        id: create-pull-request
        env:
          GH_TOKEN: ${{ secrets.AUTO_DEP_TOKEN }}
        run: |
          PULL_REQUEST_URI=$(gh pr create -B stage -t 【定期実行】dev2stage -l 'auto merge' -b "")
          echo "::set-output name=PULL_REQUEST_URI::$PULL_REQUEST_URI"
      - name: 【定期実行】Merge Pull Request dev2stage
        env:
          GH_TOKEN: ${{ secrets.AUTO_DEP_TOKEN }}
        run: |
          gh pr merge ${{steps.create-pull-request.outputs.PULL_REQUEST_URI}} --merge
```

## 定期実行

定期実行部分は、以下です。

```yaml
on:
  schedule:
    - cron: "0 15 * * *"
```

日本時間に合わせるために、15 にしています。

## プルリクエスト作成部分

```yaml
- name: 【定期実行】Create Pull Request dev2stage
  id: create-pull-request
  env:
    GH_TOKEN: ${{ secrets.AUTO_DEP_TOKEN }}
  run: |
    PULL_REQUEST_URI=$(gh pr create -B stage -t 【定期実行】dev2stage -l 'auto merge' -b "")
    echo "::set-output name=PULL_REQUEST_URI::$PULL_REQUEST_URI"
```

プルリクエスト作成部分では、[GitHub CLI](https://cli.github.com/)を使用して、プルリクエストを作成します。
`gh pr create` 部分で、CLI を用いてプルリクエストの作成を行っています。
[gh pr コマンド](https://cli.github.com/manual/gh_pr_create)のオプションで、ベースブランチ、タイトル、ラベル、内容を設定しています。

この箇所でエラーがでる場合が、いくつかあります。

1. すでに`dev`->`stage`のプルリクエストが存在している場合
1. コンフリクトが存在する場合
1. 差分が存在しない場合

以上 3 つが、エラーでプルリクエストが作成できない場合です。

## マージ部分

```yaml
- name: 【定期実行】Merge Pull Request dev2stage
  env:
    GH_TOKEN: ${{ secrets.AUTO_DEP_TOKEN }}
  run: |
    gh pr merge ${{steps.create-pull-request.outputs.PULL_REQUEST_URI}} --merge
```

マージ部分では、[GitHub CLI](https://cli.github.com/)を使用して、マージします。

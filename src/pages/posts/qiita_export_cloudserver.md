---
layout: ../../layouts/MarkdownPostLayout.astro
title: "クラウドサーバーを作る"
pubDate: 2022-12-03
description: "クラウドサーバーを作る"
author: "@ekusiadadus"
image:
  url: "https://avatars.githubusercontent.com/u/70436490?s=400&u=a714da7802c65046265c6848887eecddfc58b5c0&v=4"
  alt: "クラウドサーバーを作る"
tags: ["RaspberryPi", "nextcloud", "ubuntu20.04"]
---

# クラウドサーバーを作る

この記事では、Raspberry Pi4、NextCloud を使ってクラウドサーバーを作成します。

## 環境

| 環境                  |
| --------------------- |
| Raspberry Pi 4 modelB |
| Ubuntu 20.04          |
| NextCloud             |

## §1.Raspberry Pi をセットアップする

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/e533a117-1679-5d87-2eeb-1a128ced85f8.png)

### §1.1Ubuntu をインストールする

Raspberry Pi Imager を使って、Ubuntu LTS 2020 を MicorSD に入れます。
Imager で、フォーマット等ができます。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/016b444d-3bb8-c5e0-1c4e-1c600168cddb.png)

**フォーマット**
(Rasberry Pi Imager を使うと、フォーマットもしてくれます)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/2a66700d-c1ff-f4cf-3f2c-c90f8e284ccc.png)

### §1.2Ubuntu を起動する

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/01ec1fe5-7d22-30ad-1639-61a9e77ce564.png)

ラズパイ 4 はデスクトップ向け Ubuntu でもサクサク動きます。

### §1.3 アップデート

![raspi2020126_4.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/07b64777-13ff-b627-5626-8c793c38348f.gif)

## §2.クラウド構築

### §2.1 NextCloud インストール

```
sudo snap install nextcloud
```

![raspi10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/ddb9336e-d39d-8695-0c82-11b7d7e01805.png)

### §2.2 管理アカウント設定

```
sudo nextcloud.manual-install username password
```

## §3.使ってみる

Raspberry Pi の IP アドレスを確認して、同じネットワーク内にある別 PC から web ブラウザで見ると Next Cloud を使用できるようになります。

```
http://ラズパイのIPアドレス
```

Raspberry Pi の IP アドレスを固定したり、ドメインを設定することで外部からでもクラウドサーバーに接続できるようになります。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/35acb465-836d-14b4-6ce0-947e3efa093a.png)

## §4.おまけ(ファイルサーバーを作る)

### Samba インストール

```
sudo apt install samba samba-common-bin
```

### Samba の config を変える

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/2df3c2f8-080c-6288-4999-0fbc322dbb86.png)

```
[share]
path = /home/share
browseable = yes
writable = yes
省略
```

### Share フォルダを作る

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/905557/4ffebd80-b908-ba04-b016-da142fcca54c.png)

### Samba 再起動

```
sudo systemctl restart samba
```

## §5.おまけ(ラズパイに ssh 接続(Windows))

```
ssh RaspberryPiユーザ名@IPアドレス
```

## §5.終わりに

いかがでしたでしょうか？
Raspberry Pi を使うと、自宅でサーバー周りについて、手を動かしながら学べるのでとても楽しいです。

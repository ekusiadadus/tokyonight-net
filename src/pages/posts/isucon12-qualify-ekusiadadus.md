---
layout: ../../layouts/MarkdownPostLayout.astro
title: "ISUCON12予選にGo言語で出場して、26位で予選落ちした話"
pubDate: 2022-12-03
description: "ISUCON12予選にGo言語で出場して、26位で予選落ちした話"
author: "@ekusiadadus"
image:
  url: "https://avatars.githubusercontent.com/u/70436490?s=400&u=a714da7802c65046265c6848887eecddfc58b5c0&v=4"
  alt: "ISUCON12予選にGo言語で出場して、26位で予選落ちした話"
tags: ["GitHub Actions", "Cron", "GitHub CLI"]
---

# ISUCON12 予選に Go 言語で出場して、26 位で予選落ちした話

## ISUCON12 予選

ISUCON12 予選にチーム「mnsys」として参加しました。

最終スコアは、`20943`で 26 位で予選落ちしてしまいました...:(

(あと一つ順位が上がれば、予選通過できた(´;ω;｀))

[スコア表](https://isucon.net/archives/56838276.html)

![](/images/isucon12q/score.png)

## タイムライン

### 9 時 45 分～

午前 9 時に起床成功

![](/images/isucon12q/1.png)

9 時 45 分くらいまで、ご飯食べたり珈琲作ったりだらだら

### 10 時～

競技開始

一回目のベンチが終わる。

初期スコア: `2910`

![](/images/isucon12q/bench1.png)

競技開始と同時に、MySQL の ER 図を取り始める。

![](/images/isucon12q/2.png)

MySQL WorkBench を使っていつも通り。

テーブルが 3 つで違和感を感じ始める。

チームメイトが、インフラ周りの設定を終わらせて netdata が見られるようになる。
ここらへんで、普段の秘伝のタレがもしかしたら使えないかもということに気づき始める。

チームメイトが、Docker 外しをする。

CRUD の整理を行い始める。
ここで、Sqlite が使用されていることに気づき焦り始める。

kataribe の解析結果が出始める。

```
# Top Query (総時間順)

 total(ms) | count | average(ms) | content
-----------+-------+-------------+-------------------------------------------------------------------------------------------------------------------------------------
110773.691 |  1741 |      63.626 | SELECT player_id, MIN(created_at) AS min_created_at FROM visit_history WHERE tenant_id = ? AND competition_id = ? GROUP BY player_id
 49125.176 |  8467 |       5.802 | REPLACE INTO id_generator (stub) VALUES (?);
  4824.950 |   765 |       6.307 | INSERT INTO visit_history (player_id, tenant_id, competition_id, created_at, updated_at) VALUES (?, ?, ?, ?, ?)
  2237.724 |  1771 |       1.264 | SELECT * FROM tenant WHERE name = ?
   680.238 |   765 |       0.889 | SELECT * FROM tenant WHERE id = ?
    49.477 |     7 |       7.068 | INSERT INTO tenant (name, display_name, created_at, updated_at) VALUES (?, ?, ?, ?)
    13.494 |    13 |       1.038 | SELECT * FROM tenant ORDER BY id DESC
```

### 11 時～

CRUD 解析がほぼほぼ終わってくる。

![](/images/isucon12q/crud.png)

この段階で、Sqlite のままいくのか MySQL に移行するのかが話題になり始める。

自分は、Sqlite をメインにチューニングを行っていき、チームメンバーが MySQL に移行していくという形で行こうという話になる。

MySQL 側で、ユニークな ID を発番している箇所を見つけて UUID にすればよくね...?になる

![](/images/isucon12q/uuid1.png)

![](/images/isucon12q/uuid2.png)

### 12 時～ 14 時くらいまで

ここら辺は、地道に N+1 クエリの解消をすすめる。
途中、BulkInsert できそうな箇所があったりしたので修正していく。

![](/images/isucon12q/bulkd1.png)

また、Nginx でのキャッシュを考えたけれどどうやらもともとキャッシュされていそうであんまり効果がなさそう。。。

### 14 時～

ここらへんで、息切れしてきて自分だけで解消できる N+1 クエリがなくなってきてどうしようになってくる
Sqlite 側のチューニングをするために、同梱されていた SqlTracer を使って解析していく

```
{"time":"2022-07-23T04:09:47Z","statement":"SELECT * FROM competition WHERE tenant_id=?","args":[15],"query_time":0.000725673,"affected_rows":0}
{"time":"2022-07-23T04:09:47Z","statement":"SELECT * FROM competition WHERE id = ?","args":["1109b4b19"],"query_time":0.000030392,"affected_rows":0}
{"time":"2022-07-23T04:09:47Z","statement":"SELECT DISTINCT(player_id) FROM player_score WHERE tenant_id = ? AND competition_id = ?","args":[15,"1109b4b19"],"query_time":0.000033338,"affected_rows":0}
{"time":"2022-07-23T04:09:47Z","statement":"SELECT * FROM competition WHERE id = ?","args":["12339ae82"],"query_time":0.000019128,"affected_rows":0}
{"time":"2022-07-23T04:09:47Z","statement":"SELECT DISTINCT(player_id) FROM player_score WHERE tenant_id = ? AND competition_id = ?","args":[15,"12339ae82"],"query_time":0.00005299,"affected_rows":0}
{"time":"2022-07-23T04:09:47Z","statement":"SELECT * FROM competition WHERE id = ?","args":["125ca9262"],"query_time":0.000017787,"affected_rows":0}
{"time":"2022-07-23T04:09:47Z","statement":"SELECT DISTINCT(player_id) FROM player_score WHERE tenant_id = ? AND competition_id = ?","args":[15,"125ca9262"],"query_time":0.000025837,"affected_rows":0}
{"time":"2022-07-23T04:09:47Z","statement":"SELECT * FROM competition WHERE id = ?","args":["16df8a920"],"query_time":0.000022203,"affected_rows":0}
```

↑ のような感じで、Sqlite のクエリごとの集計を得られた。
ここから、Sqlite にインデックスを貼っていく作業に移った(たしか)

tenantDB を Create している関数があったので、DB を作成する瞬間にインデックス貼ればよさそうになった。

![](/images/isucon12q/index1.png)

この時点で、スコア `8000`-`9000` を出せるようになった。

### 16 時～

この時間は、Sqlite から MySQL に本当に移行間に合うのかが問題になってきつつあった。

15 時 30 分くらいの時点で、MySQL への移行を済ませていたけれど実際にスコアが伸びるわけでもなく、Initalize にも時間がかかりすぎて落ちてしまうことがあったため焦る。

この時点で、2 台構成(MySQL+Sqlite)で、`20000`点を出すようになっていた。

### 17 時～

MySQL を 2 台に分散しようとすると、スコアが 0 になる...

flock も外すだけ...と思っていたけど外せない。

30 分くらいまで、0 点が続き revert して何とか動くようになるところまで巻き戻る。
このとき、スコア`18000`くらい。
16 時の時点で、スコア`20000`は出ていたのでめちゃくちゃに焦る。

最終的に、MySQL の分散をあきらめてログ等のプロファイラを外す。
再起動試験を行う。

なんとか、スコア`20943`に戻して終了。

![](/images/isucon12q/score2.png)

## まとめ

全体的に、Sqlite へどのようにアプローチしていくのかを考える ISUCON 会だったなぁという感じ。

Sqlite から MySQL に移行する判断を午前中にできたのはいいものの、その後、実力不足でスピード感をもってうごけなかったなぁという後悔。。。

チームメンバーが、Sqlite -> MySQL に移行するという天才技を行ったり、インフラ整備を完璧に進めてくれたけれど、自分が臨機応変に対応できなかったなぁと思っています。

1 か月くらい週末集まって、チームで練習できたのでとても楽しかったです！

また来年頑張ります！

![](/images/isucon12q/timeline.png)

---
title: "【XRPL徹底解説】Ripplingとは？"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["XRPL", "ブロックチェーン", "Web3"]
published: false
---
# はじめに

こんにちは！一般社団法人XRPL Japan 理事の増田（通称：ぽこ太郎）です。

今回は XRPL における通貨移動のキモ機能 **「Rippling」** を徹底解説します！

この記事では以下のポイントを紹介します：
1. Rippling の仕組み
2. NoRipple / DefaultRipple の使い方
3. 安全に使うためのポイント

# トラストライン

- XRP Ledgerのルールである「不要なトークンを他者に保有させることはできない」という原則を強制するもの
- RippleStateで定義されている
  - https://xrpl.org/ja/docs/references/protocol/ledger-data/ledger-entry-types/ripplestate

# XRPLにおける通貨とは？

XRPLでは、発行された通貨（トークン）は **「通貨コード(Currency)」と「発行者アドレス（Issuer）」の組み合わせで定義** されます。
このセット（Currency, Issuer）によって、通貨は一意に識別されます。
- 発行者の定義は、残高がマイナスとなっているアカウント

たとえば「USD」という名前の通貨でも…
- USD @ IssuerA → A が発行した USD
- USD @ IssuerB → B が発行した USD

このように、見た目が同じ通貨コードでも、 **誰が発行したかによって実質的な価値や信頼性が異なります** 。


# Ripplingとは？

> XRP Ledgerでは、「Rippling」とは同一通貨のトラストラインを有する複数の接続当事者間での非可分なネット決済のプロセスを指しています。

公式ドキュメントでは上記のように記されていますが、少し難しく感じますね。
一言で表すと「同じ通貨を信頼している人たちの間で、お金を間接的にやり取りできるXRPLの仕組み」です。

## Ripplingを使わない場合

Ripplingを使わないと、送金相手が自分と同じ発行者（Issuer）を信頼していることが必要です。

✅ 同じ発行者を信頼している場合

- Aliceは USD @ 発行者A を信頼している
- Bobも USD @ 発行者A を信頼している
- Alice → Bobに直接送金ができる

❌ 発行者が異なる or 信頼していない場合

- Aliceは USD @ 発行者B を信頼している
- Bobも USD @ 発行者B を信頼していない
- Alice → Bobに直接送金ができない

## Ripplingを使う場合

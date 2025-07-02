---
title: "XRP LEDGERのマルチチェーン機能の強化"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["XRPL", "ブロックチェーン", "Web3"]
published: true
---

# はじめに

こんにちは！一般社団法人XRPL Japan 理事をしている増田（通称：ぽこ太郎）です。

### この記事の目的

最近、XRP Ledger（XRPL）に関するマルチチェーン機能のニュースが話題となっていますね！
本記事では、この最新動向をきっかけに、 **XRPLとマルチチェーン技術の関係について学べる内容** をお届けします。

### 参考リンク集

この記事で扱う用語や関連プロジェクトについて、まずはざっくりとまとめておきます。

|カテゴリ|用語|URL|
|:--|:--|:--|
|ブロックチェーン|XRP Ledger|https://xrpl.org/|
|ブロックチェーン|XRPL EVM|https://www.xrplevm.org/|
|相互運用プロトコル|Axelar|https://www.axelar.network/|
|相互運用プロトコル|Wormhole|https://wormhole.com/|
|DApp & 開発ツール|Squid router|https://www.squidrouter.com/|


# マルチチェーンとは

「マルチチェーン」とは、 **複数のブロックチェーンネットワークが連携して動作する仕組み** のことです。

![](/images/2025-07-02-xrpl-bridge/01.png)

> XRPLで発行された資産を、XRPL EVMなど他のチェーンへ送金するイメージ

## なぜ XRPL にマルチチェーン機能？

XRPLは、 **オープンな金融インフラと機関向けアプリケーションを支える** ために設計されたブロックチェーン です。

このマルチチェーン機能を活用することで、**高速・低コスト・準拠性を保ったまま**、他のチェーンへエコシステムを展開することができます。

![](/images/2025-07-02-xrpl-bridge/02.png)

こうしたマルチチェーン対応によって、XRPLエコシステムの**柔軟性・拡張性**が大きく向上します。

## マルチチェーンが解決すること

今回の統合により、開発者は以下のようなことが実現可能になります：

1. **XRPLで発行されたトークンを、他のチェーンに送れる！**
2. **XRPLから他チェーン上のスマートコントラクトを実行できる！**

この連携により、以下のようなユースケースがマルチチェーンで実現可能になります：

- **クロスチェーン決済アプリ（Payments）**
  - EthereumのUSDCで、XRPL上のウォレットに支払い 等
- **分散型金融（DeFi）**
  - XRPL上のXRPを担保にして、他チェーンのUSDCを借りる 等
- **トークン化現実資産（RWA）**
  - 他チェーンで発行されたトークンを、XRPL上で決済して購入 等

# マルチチェーン機能を提供するプロトコル

XRPLと接続可能なマルチチェーン機能を提供するプロトコルには、以下の2つが注目されています。

|項目|Axelar|Wormhole|
|:--|:--|:--|
|**チェーン**|Axelar専用チェーンあり|Wormhole専用チェーンなし|
|**対応チェーン数**|40以上|35以上|
|**URL**|https://www.axelar.network/|https://wormhole.com/|

どちらも「マルチチェーンを実現する」という目的は同じですが、 **連携の仕組みやアーキテクチャに違い** があるのが特徴です。
※ 基本的な構成はほぼ同じで、独自のチェーンを使っているかどうかが主な違いです。

# Axelarの仕組み

Axelarの最大の特徴は、 **専用のブロックチェーン（独自のL1チェーン）** を持っている点です。
このチェーンが、送信元から送信先へのトランザクションの中継と検証の役割を担っています。

![](/images/2025-07-02-xrpl-bridge/03.png)

> AXELARSCAN: https://axelarscan.io/

## Squid Router

Squid Routerは、異なるブロックチェーン間でトークンやスマートコントラクトを簡単にやり取りできるようにするサービスです。

https://app.squidrouter.com/

### XRPL から XRPL EVMをブリッジする仕組み

#### ブリッジのステップ

1. XRPL から XRPL EVM へ XRPをブリッジを実行
2. Payment Transactionで、XRPL上の [Gateway wallet](https://livenet.xrpl.org/accounts/rfmS3zqrQrka8wVyhXifEeyTwe8AMz2Yhw) へ送金
    - Payment Transactionのメモ欄にブリッジ情報を指定
3. Gateway walletがAxelarに通知
4. Axelarチェーンのバリデーターが検証・合意してメッセージを承認
5. 承認されたメッセージが送信先チェーンのGatewayに通知
6. 送信先チェーン（XRPL EVM）で トークンがMint（発行）される

※ [AXELAR - Chains](https://axelarscan.io/resources/chains) から対応しているGateway walletの確認が可能

#### イメージ図

![](/images/2025-07-02-xrpl-bridge/04.png)

XRPL EVM上には、Axelarに準拠した Squid Router のスマートコントラクトが実装されています。
このスマートコントラクトのアドレスを、XRPLの Payment Transaction のメモ欄に指定することで、コントラクトを実行することができます。

https://explorer.testnet.xrplevm.org/address/0x9bEb991eDdF92528E6342Ec5f7B0846C24cbaB58

# Wormholeの仕組み

Wormholeの特徴は、独自のブロックチェーンを持たず、「Wormhole Core Contracts」「Guardian Network」「Relayer」という3つの仕組みによって構成されている点です。
このように独自チェーンを持たない構造から、Layer 0（L0）プロトコルとも呼ばれています。

![](/images/2025-07-02-xrpl-bridge/05.png)

> XRPL上での構成がまだ明確ではないため、現時点では Ethereum と Polygon を例に図を作成しています。

## Ripple、Wormhole統合でマルチチェーン対応インフラを強化を発表

現時点では「統合が発表された段階」であり、今後、開発が本格的に進んでいくと思います。

https://ripple.com/insights/ripple-expands-multichain-interoperability-infrastructure-with-wormhole/

# 最後に...

この記事を通じて、XRPLのマルチチェーン対応や、Axelar・Wormholeの仕組みについて少しでも理解が深まっていれば嬉しいです。

今後、Web3やブロックチェーンがチェーンの壁を越えてつながっていく世界の中で、XRPLがどんな可能性を広げていくのか、ぜひ一緒にワクワクしながら追いかけていきましょう！

最後まで読んでいただき、ありがとうございました！

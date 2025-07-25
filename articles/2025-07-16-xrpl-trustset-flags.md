---
title: "フラグ"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["XRPL"]
published: false
---
# TrustSet Flags

TrustSetタイプのトランザクションでは、Flags フィールドに以下の追加値が使用可能です。

|フラグ名|Hex値|Decimal値|説明|
|:---|:---|:---|:---|
|`tfSetfAuth`|`0x00010000`|65536|このアカウントから発行された通貨を、相手アカウントに保有させることを許可します。また、一度設定すると、解除することはできません。<br>（※この設定は、asfRequireAuthフラグが有効でないと効果はありません。）|
|`tfSetNoRipple`|`0x00020000`|131072|No Ripple フラグを有効にします。このフラグが両方のトラストラインで有効になっている場合、同じ通貨間のリップリング（中継送金）をブロックします。|
|`tfClearNoRipple`|`0x00040000`|262144|No Ripple フラグを無効にし、このトラストラインでのリップリング（中継送金）を許可します。|
|`tfSetFreeze`|`0x00100000`|1048576|このトラストラインを凍結します。|
|`tfClearFreeze`|`0x00200000`|2097152|このトラストラインの凍結を解除します。|
|`tfSetDeepFreeze`|`0x00400000`|4194304|このトラストラインをディープフリーズ（完全凍結）します。|
|`tfClearDeepFreeze`|`0x00800000`|8388608|このトラストラインのディープフリーズを解除します。|

## 仕様

- currency と issuer の組み合わせによってトークンが一意に識別される。
  - 同一の currency と issuer で TrustSet を実行した場合、新たなトラストラインが作成されるのではなく、既存のトラストラインが更新される。

https://x.com/Ripple/status/1945348210899239159

## 各フラグの説明

### tfSetfAuth

#### まず登場人物と前提

- Alice は通貨発行者（IOUの発行元）です。
- Bob はAliceの発行するトークンを受け取りたいユーザーです。
- XRP Ledger では、IOUを送受信するために 信頼線（Trust Line） を張る必要があります。

#### 🔐 asfRequireAuth：Aliceの設定

asfRequireAuth を Aliceのアカウントに設定すると、「Bobが勝手にAliceのトークンを受け取れない」ようになります。
つまり、認可された人にだけ、うちの通貨を持ってもらいます〜！
というフラグ。

#### 🧠 設定結果

- Aliceが asfRequireAuth をオンにしていれば、
  - Bobはトラストラインを作るだけでは保有できません！
- Aliceが 個別に「OKしてあげる」必要があります。

#### 🎯 処理の流れ（ステップごとに）

| ステップ | 処理内容                                   | 詳細説明                                                                 |
|:--------:|:------------------------------------------|:------------------------------------------------------------------------|
| ①        | Alice が asfRequireAuth を有効にする      | 通貨の保有に認可が必要になる（SetFlag: 2）                              |
| ②        | Bob が TrustSet を送信                    | 通常通りトラストラインを開こうとする（でも未承認なので使えない）         |
| ③        | Alice が Bob に対して TrustSet + tfSetfAuth を送信 | Bobの通貨保有を許可（SetfAuth: 0x00010000）                             |
| ④        | Bob のトラストラインがアクティブに！      | これでBobは通貨を受け取れるようになります                               |

### tfSetNoRipple / tfClearNoRipple

作成中

### tfSetFreeze / tfClearFreeze

作成中

### tfSetDeepFreeze / tfClearDeepFreeze

作成中

## 実装サンプル

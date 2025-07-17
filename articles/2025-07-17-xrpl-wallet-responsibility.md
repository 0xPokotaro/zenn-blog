---
title: "【XRPL運用ベストプラクティス】トランザクション送信における責務分離の実装指針"
emoji: "😑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["XRPL", "ブロックチェーン", "Web3"]
published: true
---
# はじめに
こんにちは！一般社団法人XRPL Japan 理事をしている増田（通称：ぽこ太郎）です。

XRPL を使った DApp の本番運用においては、ウォレットとトランザクション送信処理の **責務を整理** することが重要です。
本記事では、 **ウォレットSDKは署名のみを担い、送信処理は別レイヤに任せる設計** を通じて、保守性・拡張性を高める方法を紹介します。

# ウォレットSDKの責務は分けるべきか？

トランザクション送信において、ウォレットの SDK にどこまでの責務を持たせるべきかは、本番環境での運用設計において重要な論点です。
ここでは、 **ウォレットSDK に送信処理の責務を集約した場合** に発生する処理内容を整理し、代表的なウォレット SDK の一例として GemWallet を取り上げて考察します。

https://gemwallet.app/

# トランザクション送信における責務の整理

ウォレット SDK が送信処理全体を担う構成では、UI 層にトランザクションロジックが集中しやすく、SDK の更新対応や互換性の維持が難しくなるため、長期的な保守性や拡張性に課題が生じやすくなります。

|No|責務|役割|
|:---|:---|:---|
|1|トランザクションの作成|トランザクションに応じたパラメータ生成|
|2|署名処理|トランザクションへの署名|
|3|トランザクションの送信|ネットワークへブロードキャスト|
|4|送信結果の取得と処理|成否判定、エラー時のリカバリ制御|

# 責務の分け方と設計の考え方

ウォレット SDK の責務を適切に分離するには、トランザクション送信のライフサイクルを次の 3 つのレイヤーに分けて考えると整理しやすくなります。

![](/images/2025-07-17-xrpl-wallet-responsibility/01.png)

## 1. トランザクションの作成レイヤ

このレイヤでは、型定義やバリデーションの設計方針によって実装のアプローチが大きく変わります。
設計時には、型の保守性・将来的な拡張性・SDKへの依存度、そしてバリデーションの柔軟性などのバランスを意識することが重要です。

### 実装方法
| 区分 | 方法 | メリット | 注意点 |
|:----:|:-----|:--------|:-------|
| **松** | 独自実装した型定義を使用 | 柔軟性と型安全性に優れ、長期運用に強い | 初期コストは高め |
| **竹** | `xrpl.js` を使用 | 最新の仕様に追随しやすく、広く使われている | 更新や破壊的変更に注意が必要 |
| **梅** | ウォレットSDK を使用 | 実装が簡易で取り組みやすい | 柔軟性と拡張性は限定される |

> 💡 個人的には、保守性と拡張性のバランスが良いため 竹（xrpl.js の型定義を直接使用） を選択します。

サンプルコード

```typescript
import { signTransaction, getAddress } from '@gemwallet/api'
import type { SubmittableTransaction } from 'xrpl' // ここで xrpl.js を使用

async function sendTrustSetTransaction() {
  const res = await getAddress()

  // xrpl.jsの型定義を使用
  const transaction: SubmittableTransaction = {
    TransactionType: 'TrustSet',
    Account: res.result.address,
    LimitAmount: {
      currency: 'ABC',
      issuer: 'rLQ23456789012345678901234567890123456789',
      value: '100'
    },
  }
}
```

## 2. 署名レイヤ

署名レイヤでは、ユーザーの秘密鍵を用いて安全に署名を行うことが主な責務です。
ウォレット SDK を通じて署名するため、インターフェースの抽象化と対応ウォレットごとの挙動を統一的に扱うことが重要です。

### 実装方法
| 区分 | 方法 | メリット | 注意点 |
|:----:|:-----|:--------|:-------|
| - | ウォレットSDK を使用 | 実装は非常に簡易 | マルチウォレット対応で複雑化しやすい |

> 💡 マルチウォレットに対応する場合は、ウォレット種別ごとにファクトリーを用いて適切な SDK 実装を生成する構成を取ると、実装の切り替えや拡張がスムーズになります。

サンプルコード

```typescript
import { signTransaction, getAddress } from '@gemwallet/api'  // ここで ウォレットSDK を使用
import type { SubmittableTransaction } from 'xrpl'

async function sendTrustSetTransaction() {
  const res = await getAddress()

  const transaction: SubmittableTransaction = {
    TransactionType: 'TrustSet',
    Account: res.result.address,
    LimitAmount: {
      currency: 'ABC',
      issuer: 'rLQ23456789012345678901234567890123456789',
      value: '100'
    },
  }

  // ウォレットSDKを使用して署名
  const response = await signTransaction({ transaction })
  const txBlob = response?.result?.signature

  console.log('txBlob: ', txBlob)
}
```


## 3. 送信・監視レイヤ

送信・監視レイヤでは、署名済みトランザクションのネットワーク送信とその結果の監視・記録を担います。

### 実装方法
| 区分 | 方法 | メリット | 注意点 |
|:----:|:-----|:--------|:-------|
| **松** | Backend API 経由で送信 | セキュリティ・可観測性が高い | サーバーインフラの構築と保守、実装コストが大きい |
| **竹** | フロントエンドから WSS 経由で直接送信 | レイテンシが低く、UXに優れている | 接続管理やセキュリティ要件を別途設計する必要がある |
| **梅** | ウォレットSDK を使用 | 実装が簡易で取り組みやすい | 柔軟性と拡張性は限定される |

> 💡 私ならフロントエンドとバックエンドを TypeScript で統一したモノリシック構成を採用し、**松（Backend API 経由）**で送信処理を実行します。これにより、セキュリティと可観測性を高めつつ、保守性の高い実装が可能になります。

サンプルコード

```typescript
// バックエンド(NextJS)で、トランザクションを送信
const response = await fetch(XRPL_API_URL, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    method: 'submit',
    params: [
      {
        tx_blob: txBlob
      }
    ]
  })
})

const json = await response.json()
const result = json.result

console.log('result: ', result)
```


# まとめ

ウォレット SDK に送信処理をすべて任せる設計は、一見シンプルですが、保守や拡張の面で課題が多くなりがちです。
そのため、本記事では処理を「リクエスト作成」「署名」「送信・監視」の3つのレイヤーに分け、それぞれの責務に応じた最適な実装方法を整理しました。

- **責務を明確に分離することで、保守性・拡張性・セキュリティ・可観測性を高められる**
- **各レイヤーの実装方針は、開発体制やプロダクトのフェーズに応じて柔軟に選択するのが重要**

# 最後に...

今後、Web3やブロックチェーンがチェーンの壁を越えてつながっていく世界の中で、XRPLがどんな可能性を広げていくのか、ぜひ一緒にワクワクしながら追いかけていきましょう！

最後まで読んでいただき、ありがとうございました！

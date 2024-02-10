---
title: Lexicon（辞書）
summary: スキーマ駆動の相互運用性フレームワーク
tldr:
 - Lexiconはグローバルなスキーマシステムです
 - com.example.ping()のような逆DNS名を使用します
 - 定義はJSONドキュメントで、JSON-Schemaに似ています
 - 現在はHTTPエンドポイント、イベントストリーム、およびリポジトリレコードに使用されています
---

# Lexicon入門

LexiconはRPCメソッドとレコードタイプを定義するためのスキーマシステムです。各Lexiconスキーマは、[JSON-Schema](https://json-schema.org/)と似た形式のJSONで書かれています。

これらのスキーマは、逆DNS形式の[NSID](/specs/nsid)を使用して識別されます。以下はいくつかの例です：

```typescript
com.atproto.repo.getRecord()
com.atproto.identity.resolveHandle()
app.bsky.feed.getPostThread()
app.bsky.notification.listNotifications()
```

そして、いくつかの例のレコードタイプ：

```typescript
app.bsky.feed.post
app.bsky.feed.like
app.bsky.actor.profile
app.bsky.graph.follow
```

スキーマのタイプ、定義言語、およびバリデーションの制約については、[Lexicon仕様](/specs/lexicon)で説明されており、JSONとCBORでの表現は[データモデル仕様](/specs/data-model)で説明されています。

## Lexiconが必要な理由

**相互運用性。** atprotoのようなオープンネットワークは、振る舞いとセマンティクスに合意する方法が必要です。 Lexiconはこれを解決し、新しいスキーマを導入する際に開発者に対して比較的簡単にします。

**LexiconはRDFではありません。** RDFはデータを記述するのに効果的ですが、スキーマを強制するには理想的ではありません。 LexiconはRDFが提供する一般性が不要なため、使用が容易です。実際、Lexiconのスキーマは型とバリデーションのコード生成を可能にし、これにより開発が大幅に簡素化されます。

## HTTP APIメソッド

ATプロトコルのAPIシステムである[XRPC](/specs/xrpc)は、基本的にはHTTPSの薄いラッパーです。その目的は、LexiconをHTTPSに適用することです。

例えば、以下の呼び出し：

```typescript
com.example.getProfile()
```

実際にはただのHTTPリクエストです：

```text
GET /xrpc/com.example.getProfile
```

スキーマはクエリパラメータ、リクエストボディ、およびレスポンスボディの有効なものを確立します。

```json
{
  "lexicon": 1,
  "id": "com.example.getProfile",
  "type": "query",
  "parameters": {
    "user": {"type": "string", "required": true}
  },
  "output": {
    "encoding": "application/json",
    "schema": {
      "type": "object",
      "required": ["did", "name"],
      "properties": {
        "did": {"type": "string"},
        "name": {"type": "string"},
        "displayName": {"type": "string", "maxLength": 64},
        "description": {"type": "string", "maxLength": 256}
      }
    }
  }
}
```

コード生成を使用すると、これらのスキーマは非常に使いやすくなります：

```typescript
await client.com.example.getProfile({user: 'bob.com'})
// => {name: 'bob.com', did: 'did:plc:1234', displayName: '...', ...}
```

## レコードタイプ

スキーマはレコードの可能な値を定義します。各レコードには「タイプ」があり、それはスキーマにマッピングされ、またレコードのURLを確立します。

例えば、この「follow」レコード：

```json
{
  "$type": "com.example.follow",
  "subject": "at://did:plc:12345",
  "createdAt": "2022-10-09T17:51:55.043Z"
}
```

...は次のようなURLを持ちます：

```text
at://bob.com/com.example.follow/12345
```

...および次のようなスキーマ：

```json
{
  "lexicon": 1,
  "id": "com.example.follow",
  "type": "record",
  "description": "A social follow",
  "record": {
    "type": "object",
    "required": ["subject", "createdAt"],
    "properties": {
      "subject": { "type": "string" },
      "createdAt": {"type": "string", "format": "datetime"}
    }
  }
}
```

## トークン

トークンはデータで使用できるグローバルな識別子を宣言します。

例えば、レコードスキーマが交通信号の3つの可能な状態を指定したい場合を考えてみましょう：'red'、'yellow'、および'green'。

```json
{
  "lexicon": 1,
  "id": "com.example.trafficLight",
  "type": "record",
  "record": {
    "type": "object",
    "required": ["state"],
    "properties": {
      "state": { "type": "string", "enum": ["red", "yellow", "green"] },
    }
  }
}
```

これは完璧に受け入れられるものですが、拡張可能ではありません。新しい状態を追加することはできません。例えば、「flashing yellow」や「purple」といった新しい状態を追加することはできません。

柔軟性を追加するために、enumの制約を削除し、可能な値を文書化するだけでもよいです：

```json
{
  "lexicon": 1,
  "id": "com.example.trafficLight",
  "type": "record",
  "record": {
    "type": "object",
    "required": ["state"],
    "properties": {
      "state": {
        "type": "string",
        "description": "Suggested values: red, yellow, green"
      },
    }
  }
}
```

これは悪くありませんが、具体性が不足しています。状態の新しい値を発明する人々はお互いに衝突しやすく、各状態についての明確な文書がないかもしれません。

代わりに、使用する値のためにLexiconトークンを定義できます：

```json
{
  "lexicon": 1,
  "id": "com.example.green",
  "type": "token",
  "description": "Traffic light state representing 'Go!'.",
}
{
  "lexicon": 1,
  "id": "com.example.yellow",
  "type": "token",
  "description": "Traffic light state representing 'Stop Soon!'.",
}
{
  "lexicon": 1,
  "id": "com.example.red",
  "type": "token",
  "description": "Traffic light state representing 'Stop!'.",
}
```

これにより、trafficLightの状態で使う明確な値が得られます。最終的なスキーマは柔軟なバリデーションを使用しますが、他のチームは値の起源と独自の値を追加する方法についてより明確に理解できます：

```json
{
  "lexicon": 1,
  "id": "com.example.trafficLight",
  "type": "record",
  "record": {
    "type": "object",
    "required": ["state"],
    "properties": {
      "state": {
        "type": "string",
        "knownValues": [
          "com.example.green",
          "com.example.yellow",
          "com.example.red"
        ]
      },
    }
  }
}
```

## バージョニング

一度スキーマが公開されると、制約を変更することはできません。制約を緩める（可能な値を追加する）と、古いソフトウェアが新しいデータの検証に失敗します。制約を強化する（可能な値を削除する）と、新しいソフトウェアが古いデータの検証に失敗します。結果として、スキーマは以前に制約のないフィールドにのみオプションの制約を追加できます。

スキーマは機械可読でネットワークアクセス可能に設計されています。現在はネットワーク上でスキーマが利用可能である必要がないため、スキーマを公開することが必須ではありませんが、メソッドの消費者が単一の正規および信頼性のある表現が利用可能であるようにするために強く勧められています。

## スキーマの配布

スキーマは機械可読でネットワークアクセス可能なように設計されています。現在はネットワーク上でスキーマが利用可能であることが _必須ではありません_ が、メソッドのコンシューマに対して単一の正規で信頼性のある表現が利用可能であるように、スキーマを公開することが強く勧められています。

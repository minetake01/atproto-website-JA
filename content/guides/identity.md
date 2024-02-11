---
title: アイデンティティ
summary: ATプロトコルがユーザーのアイデンティティをどのように扱うか
tldr:
  - 各ユーザーはalice.host.comや単にalice.comのようなドメイン名のハンドルを持っています
  - 各ユーザーはマイグレーションを可能にする永続的な「DID」も持っています
  - DIDはユーザーのキーと現在のホストアドレスにマップされます
---

# アイデンティティ

atprotoのアイデンティティシステムにはいくつかの要件があります：

* **IDの提供.** ユーザーはサービスを横断して安定したグローバルIDを作成できるようにする必要があります。これらのIDは安定したコンテンツへのリンクを確保するために滅多に変更されません。
* **公開鍵の配布.** 分散システムではデータの真正性を証明し、エンドツーエンドのプライバシーを提供するために暗号技術が必要です。アイデンティティシステムは強力なセキュリティで公開鍵を公開する必要があります。
* **鍵の回転.** ユーザーはアイデンティティを妨げることなくキーマテリアルを回転させることができる必要があります。
* **サービスの検出.** アプリケーションはユーザーが使用するサービスを検出できる必要があります。
* **使いやすさ.** ユーザーは人間が読みやすく覚えやすい名前を持っているべきです。
* **移植性.** アイデンティティはサービスを横断して移植できる必要があります。プロバイダを変更してもユーザーがアイデンティティ、ソーシャルグラフ、またはコンテンツを失わないようにする必要があります。

このシステムの採用により、アプリケーションにはエンドツーエンドの暗号化、署名付きユーザーデータ、サービスへのサインイン、および一般的な相互運用のツールが提供されるはずです。

## 識別子

私たちは2つの相互関連する識別子の形式を使用しています：_ハンドル_と_DID_。ハンドルはDNS名で、DIDは[新興のW3C標準](https://www.w3.org/TR/did-core/)で、安全で安定したIDとして機能します。

以下はすべて有効なユーザー識別子です：

<pre><code>alice.host.com
at://alice.host.com
at://did:plc:bv6ggog3tya2z3vxsub7hnal
</code></pre>

それらの関係は次のように視覚化できます：

<pre style="line-height: 1.2;"><code>┌──────────────────┐                 ┌───────────────┐
│ DNS名             ├──resolves to──→ │ DID           │
│ (alice.host.com) │                 │ (did:plc:...) │
└──────────────────┘                 └─────┬─────────┘
       ↑                                   │
       │                               resolves to
       │                                   │
       │                                   ↓
       │                            ┌───────────────┐
       └───────────references───────┤ DID Document  │
                                    │ {"id":"..."}  │
                                    └───────────────┘
</code></pre>

DNSハンドルはユーザー向けの識別子であり、UIに表示され、ユーザーを見つける方法として推奨されるべきです。アプリケーションはハンドルをDIDに解決し、その後DIDを安定した正規の識別子として使用します。そのDIDは安全にDIDドキュメントに解決され、公開鍵とユーザーサービスが含まれます。

<table>
  <tr>
   <td><strong>ハンドル</strong>
   </td>
   <td>ハンドルはDNS名です。[com.atproto.identity.resolveHandle()](/lexicons/com-atproto) XRPCメソッドを使用して解決され、DIDドキュメントの一致するエントリで確認される必要があります。[ハンドルの仕様](/specs/handle)で詳細があります。
   </td>
  </tr>
  <tr>
   <td><strong>DID</strong>
   </td>
   <td>DIDは[新興のW3C標準](https://www.w3.org/TR/did-core/)で、安定した＆安全なIDを提供するためのものです。これらはユーザーの安定した正規IDとして使用されます。ATプロトコルでの使用方法の詳細は[DIDの仕様](/specs/did)にあります。
   </td>
  </tr>
  <tr>
   <td><strong>DIDドキュメント</strong>
   </td>
   <td>
    DIDドキュメントはDIDレジストリによってホストされる標準化されたオブジェクトです。以下の情報が含まれています：
    <ul>
      <li>DIDに関連するハンドル。</li>
      <li>署名鍵。</li>
      <li>ユーザーのPDSのURL。</li>
    </ul>
   </td>
  </tr>
</table>

## DIDメソッド

[DID標準](https://www.w3.org/TR/did-core/)はDIDを[DIDドキュメント](https://www.w3.org/TR/did-core/#core-properties)に公開し、解決するためのカスタム「メソッド」をサポートしています。既存のさまざまなメソッドが[公開されています](https://w3c.github.io/did-spec-registries/#did-methods)ので、この提案への含まれるための基準を確立する必要があります：

- **強力な一貫性.** 特定のDIDに対して解決クエリは常に一度に1つの有効なドキュメントを生成する必要があります（一部のネットワークでは確率論的トランザクションの確定の対象となる場合があります）。
- **高可用性.** 解決クエリは確実に成功する必要があります。
- **オンラインAPI.** クライアントは標準のAPIを介して新しいDIDドキュメントを公開できる必要があります。
- **セキュア.** ネットワークはそのオペレータ、MITM、および他のユーザーからの攻撃に対して保護する必要があります。
- **低コスト.** DIDドキュメントの作成および更新はサービスとユーザーに手頃である必要があります。
- **鍵の回転.** ユーザーはアイデンティティを失うことなくキーペアを回転させることができる必要があります。
- **分散ガバナンス.** ネットワークは単一の利害関係者によって統治されていてはならず、オープンネットワークまたはプロバイダのコンソーシアムである必要があります。

現在、いずれのDIDメソッドも完全に私たちの基準を満たしていません。**したがって、私たちは[did-web](https://w3c-ccg.github.io/did-method-web/)と[一時的なメソッドDID Placeholder](https://github.com/bluesky-social/did-method-plc)をサポートすることにしました。** この状況は新しい解決策が現れるにつれて進化すると期待されています。

## ハンドルの解決

atprotoでのハンドルはDIDに解決され、ユーザーの署名公開鍵とホスティングサービスを含むDIDドキュメントに解決されるドメイン名です。

ハンドルの解決には[`com.atproto.identity.resolveHandle`](/lexicons/com-atproto) XRPCメソッドが使用されます。メソッド呼び出しはハンドルによって識別されるサーバに送信され、ハンドルはパラメータとして渡されるべきです。

以下は擬似TypeScriptでのアルゴリズムです：

```typescript
async function resolveHandle(handle: string) {
  const origin = `https://${handle}`
  const res = await xrpc(origin, 'com.atproto.identity.resolveHandle', {handle})
  assert(typeof res?.did === 'string' && res.did.startsWith('did:'))
  return res.did
}
```

### 例: ホスティングサービス

ホスティングサービスがPLCを使用しており、ユーザーのハンドルをサブドメインとして提供しているシナリオを考えてみましょう：

- ハンドル： `alice.pds.com`
- DID： `did:plc:12345`
- ホスティングサービス： `https://pds.com`

最初は`alice.pds.com`しかわかりませんので、`com.atproto.identity.resolveHandle()`を`alice.pds.com`で呼び出します。これによりDIDがわかります。

```typescript
await xrpc.service('https://alice.pds.com').com.atproto.identity.resolveHandle() // => {did: 'did:plc:12345'}
```

次に、返されたDIDに対してPLC解決メソッドを呼び出して、ホスティングサービスのエンドポイントとユーザーのキーマテリアルを学びます。

```typescript
await didPlc.resolve('did:plc:12345') /* => {
  id: 'did:plc:12345',
  alsoKnownAs: `https://alice.pds.com`,
  verificationMethod: [...],
  service: [{serviceEndpoint: 'https://pds.com', ...}]
}*/
```

これで`https://pds.com`と通信してAliceのデータにアクセスできます。

### 例: 別のドメイン名を持つホスティングサービス

前述のシナリオと同じであると仮定しましょうが、ユーザーが独自のドメイン名を提供しているとします：

- ハンドル： `alice.com`（これは前と異なります）
- DID： `did:plc:12345`
- ホスティングサービス： `https://pds.com`

`alice.com`で`com.atproto.identity.resolveHandle()`を呼び出してDIDを取得します。

```typescript
await xrpc.service('https://alice.com').com.atproto.identity.resolveHandle() // => {did: 'did:plc:12345'}
```

次に、前と同様にDIDを解決します：

```typescript
await didPlc.resolve('did:plc:12345') /* => {
  id: 'did:plc:12345',
  alsoKnownAs: `https://alice.com`,
  verificationMethod: [...],
  service: [{serviceEndpoint: 'https://pds.com', ...}]
}*/
```

これで`https://pds.com`と通信してAliceのデータにアクセスできます。`https://alice.com`のエンドポイントは、`com.atproto.identity.resolveHandle()`の呼び出しを処理するためだけに使われます。実際のユーザーデータは`pds.com`にあります。

### 例: セルフホスティング

セルフホスティングのシナリオを考えてみましょう。それが`did:plc`を使用している場合、次のようになります：

- ハンドル： `alice.com`
- DID： `did:plc:12345`
- ホスティングサービス： `https://alice.com`

ただし、**セルフホストがドメイン名の所有権を確実に保持すると信じている場合**、`did:plc`の代わりに`did:web`を使用できます：

- ハンドル： `alice.com`
- DID： `did:web:alice.com`
- ホスティングサービス： `https://alice.com`

`alice.com`で`com.atproto.identity.resolveHandle()`を呼び出してDIDを取得します。

```typescript
await xrpc.service('https://alice.com').com.atproto.identity.resolveHandle() // => {did: 'did:web:alice.com'}
```

その後、`did:web`を使用して解決します：

```typescript
await didWeb.resolve('did:web:alice.com') /* => {
  id: 'did:web:alice.com',
  alsoKnownAs: `https://alice.com`,
  verificationMethod: [...],
  service: [{serviceEndpoint: 'https://alice.com', ...}]
}*/
```

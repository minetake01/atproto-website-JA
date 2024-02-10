---
title: よくある質問（FAQ）
summary: 認証済み転送プロトコル（ATP）に関するよくある質問
---

# よくある質問（FAQ）

認証済み転送プロトコル（ATP）に関するよくある質問。Blueskyに関するFAQは[こちら](https://blueskyweb.xyz/faq)を参照してください。

## ATプロトコルはブロックチェーンですか？

いいえ。ATプロトコルは[連邦プロトコル](https://en.wikipedia.org/wiki/Federation_(information_technology))です。これはブロックチェーンではなく、ブロックチェーンを使用しません。

## なぜActivityPubを使用しないのですか？

[ActivityPub](https://en.wikipedia.org/wiki/ActivityPub)は、[Mastodon](https://joinmastodon.org/)によって普及した連邦型ソーシャルネットワーキング技術です。

アカウントの移植性が、われわれが独自のプロトコルを構築する理由の主な要因です。移植性は、ユーザーを急なBANやサーバーシャットダウン、ポリシーの不一致から保護するために重要だと考えています。 移植性のための当社のソリューションには、[署名付きデータリポジトリ](/guides/data-repos)と[DID（分散型識別子）](/guides/identity)の両方が必要であり、これらはActivityPubに簡単に後付けできません。 ActivityPubのための移行ツールは比較的限られており、元のサーバーがリダイレクトを提供することを要求し、ユーザーの以前のデータを移行できません。

その他の細かい違いには、スキーマの取り扱いに関する異なる視点、APの二重@メールユーザー名よりもドメインユーザー名を好むこと、そしてActivityPubが好むハッシュタグスタイルの探索ではなく、大規模な検索と発見を目指している点などがあります。

## JSON-LDやRDFを使用せずにLexiconを作成する理由は何ですか？

Atprotoは組織間でデータとRPCコマンドを交換します。データとRPCが有用であるためには、ソフトウェアが異なるチームによって作成されたスキーマを正しく処理できる必要があります。これが[Lexicon](/guides/lexicon)の目的です。

私たちはエンジニアが新しいスキーマを使用し、作成する際に快適に感じ、開発者がシステムのDXを楽しむことを望んでいます。 Lexiconは、開発者に非常に馴染みのある強力な型付きAPIを生成し、分散システムで非常に重要な実行時の正確性チェックを提供します。

[RDF](https://en.wikipedia.org/wiki/Resource_Description_Framework)は、システムが非常に少ないインフラを共有している極めて一般的なケースを想定しています。概念的には優雅ですが、理解しづらい構文をしばしば追加し、開発者が理解できないことがよくあります。 JSON-LDはRDFボキャブラリを消費するタスクを単純化しますが、それはRDFをより判読可能にするのではなく、その基本的な概念を隠しています。

私たちはRDFを使用する可能性を非常に注意深く検討しましたが、開発者の体験（DX）や提供されるツールに魅力を感じませんでした。

## "XRPC"とは何で、なぜ___を使用しないのですか？

[XRPC](/specs/xrpc)は、いくつかの追加の慣習を備えたHTTPです。

XRPCは[Lexicon](/guides/lexicon)を使用してHTTPコールを記述し、それらを`/xrpc/{methodId}`にマップします。例えば、このAPIコール：

```typescript
await api.com.atproto.repo.listRecords({
  user: 'alice.com',
  collection: 'app.bsky.feed.post'
})
```

...は次のようにマップされます：

```text
GET /xrpc/com.atproto.repo.listRecords
  ?user=alice.com
  &collection=app.bsky.feed.post
```

Lexiconは共有メソッドID (`com.atproto.repo.listRecords`) と期待されるクエリパラメータ、入力ボディ、出力ボディを確立します。Lexiconを使用することで、コールの入力と出力に対する実行時のチェックを得ることができ、上記のAPIコールのような型付きのコードを生成できます。
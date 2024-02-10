---
title: アプリケーションモデル
summary: ATプロトコル上でアプリケーションが動作する仕組み
tldr:
  - アプリはユーザーのPDSにサインインしてアカウントにアクセスする
  - アプリは直接レポのレコードを読み書きできる
  - ほとんどの対話は高レベルのレキシコンを介して行われる
---

# アプリケーションモデル

ATプロトコル上のアプリケーションは、ユーザーの個人データサーバー（PDS）に接続してアカウントにアクセスします。セッションが確立されると、アプリはPDSによって実装された[レキシコン](/lexicon)を使用して動作を制御できます。

このガイドでは、いくつかの一般的なパターン（簡単なコード例付き）を進めて、これについて直感を身につけるのに役立ちます。以下に示すすべてのAPIは、Lexiconのコード生成CLIを使用して生成されています。

## サインイン

サインインと認証はシンプルなセッション指向のプロセスです。[com.atproto.server lexicon](/lexicons/com-atproto)には、これらのセッションを作成および管理するためのAPIが含まれています。

```typescript
// 自分のPDSでAPIインスタンスを作成
const api = AtpApi.service('my-pds.com')

// 識別子とパスワードを使用してサインイン
const res = await api.com.atproto.server.createSession({
  identifier: 'alice.host.com',
  password: 'hunter2'
})

// 将来の呼び出しにトークンを含めるように設定
api.setHeader('Authorization', `Bearer ${res.data.accessJwt}`)
```

## レポ CRUD

すべてのユーザーにはパブリックデータリポジトリがあります。アプリケーションはAPIを使用してレコードに対して基本的なCRUD操作を行うことができます。

```typescript
await api.com.atproto.repo.listRecords({
  repo: 'alice.com',
  collection: 'app.bsky.post'
})
await api.com.atproto.repo.getRecord({
  repo: 'alice.com',
  collection: 'app.bsky.post',
  rkey: '3jyfrk3olgd2h'
})
await api.com.atproto.repo.createRecord({
  repo: 'alice.com',
  collection: 'app.bsky.post'
}, {
  text: 'Second post!',
  createdAt: (new Date()).toISOString()
})
await api.com.atproto.repo.putRecord({
  repo: 'alice.com',
  collection: 'app.bsky.post',
  rkey: '3jyfrk3olgd2h'
}, {
  text: 'Hello universe!',
  createdAt: originalPost.data.createdAt
})
await api.com.atproto.repo.deleteRecord({
  repo: 'alice.com',
  collection: 'app.bsky.post',
  rkey: '3jyfrk3olgd2h'
})
```

上記のレポはドメイン名`alice.com`で識別されていることに気付くかもしれません。詳細については[Identity guide](/identity)を参照してください。

## レコードタイプ

"タイプ"フィールドに注目している場合、それがどのように動作するかについては[Intro to Lexicon guide](/lexicon)を参照してください。以下はATPソフトウェアで現在使用されているいくつかのタイプの簡単なリストです。

いくつかのスキーマで "cids" が使用されていることに気付くかもしれません。 "cid"は "Content ID" の略で、参照されたコンテンツのsha256ハッシュです。これらは整合性を確保するために使用されます。たとえば、いいねにはいいね対象の投稿のcidが含まれており、将来の編集が検出され、UIに表示されます。

### <a href="/lexicons/app-bsky">app.bsky.graph.follow</a>

ソーシャルフォロー。例:

```typescript
{
  $type: 'app.bsky.graph.follow',
  subject: 'did:plc:bv6ggog3tya2z3vxsub7hnal',
  createdAt: '2022-10-10T00:39:08.609Z'
}
```

### <a href="/lexicons/app-bsky">app.bsky.feed.like</a>

コンテンツのいいね。例:

```typescript
{
  $type: 'app.bsky.feed.like',
  subject: {
    uri: 'at://did:plc:bv6ggog3tya2z3vxsub7hnal/app.bsky.post/1',
    cid: 'bafyreif5lqnk3tgbhi5vgqd6wy5dtovfgndhwta6bwla4iqaohuf2yd764'
  }
  createdAt: '2022-10-10T00:39:08.609Z'
}
```

### <a href="/lexicons/app-bsky">app.bsky.feed.post</a>

マイクロブログの投稿。例:

```typescript
{
  $type: 'app.bsky.feed.post',
  text: 'Hello, world!',
  createdAt: '2022-10-10T00:39:08.609Z'
}
```

### <a href="/lexicons/app-bsky">app.bsky.actor.profile</a>

ユーザープロファイル。例:

```typescript
{
  $type: 'app.bsky.actor.profile',
  displayName: 'Alice',
  description: 'A cool hacker'
}
```

### <a href="https://atproto.com/lexicons/app-bsky">app.bsky.feed.repost</a>

既存のマイクロブログ投稿のリポスト（リツイートに類似）。例:

```typescript
{
  $type: 'app.bsky.feed.repost',
  subject: {
    uri: 'at://did:plc:bv6ggog3tya2z3vxsub7hnal/app.bsky.post/3jyfrk3olgd2h',
    cid: 'bafyreif5lqnk3tgbhi5vgqd6wy5dtovfgndhwta6bwla4iqaohuf2yd764'
  }
  createdAt: '2022-10-10T00:39:08.609Z'
}
```

## ソーシャルAPI

レポCRUDや他の低レベルの`com.atproto.*` APIでできることはたくさんありますが、`app.bsky.*`レキシコンはソーシャルアプリケーションに対してより強力で使いやすいAPIを提供しています。

```typescript
await api.app.bsky.feed.getTimeline()
await api.app.bsky.feed.getAuthorFeed({author: 'alice.com'})
await api.app.bsky.feed.getPostThread({uri: 'at://alice.com/app.bsky.post/3jyfrk3olgd2h'})
await api.app.bsky.feed.getLikes({uri: 'at://alice.com/app.bsky.post/3jyfrk3olgd2h'})
await api.app.bsky.feed.getRepostedBy({uri: 'at://alice.com/app.bsky.post/3jyfrk3olgd2h'})
await api.app.bsky.actor.getProfile({actor: 'alice.com'})
await api.app.bsky.graph.getFollowers({actor: 'alice.com'})
await api.app.bsky.graph.getFollows({actor: 'alice.com'})
await api.app.bsky.notification.listNotifications()
await api.app.bsky.notification.getUnreadCount()
await api.app.bsky.notification.updateSeen()
```
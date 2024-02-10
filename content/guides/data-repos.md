---
title: パーソナルデータリポジトリ
summary: AT Protocol リポジトリ構造のガイド
tldr:
  - データリポジトリは署名されたデータのコレクションです
  - これらはデータベースレコードのためのGitリポジトリのようなものです
  - ユーザーはこれらのリポジトリに公開活動データを追加します
---

# データリポジトリ

データリポジトリは単一のユーザーによって公開されたデータのコレクションです。リポジトリは自己認証データ構造であり、各更新は署名され、誰でも検証できる仕組みです。

これについての詳細は[リポジトリ仕様](/specs/repository)で説明されています。

## データレイアウト

リポジトリの内容は[Merkle Search Tree (MST)](https://hal.inria.fr/hal-02303490/document)に配置され、状態を単一のルートハッシュに削減します。以下のレイアウトとして視覚化できます：

<pre style="line-height: 1.2;"><code>┌────────────────┐
│     Commit     │  (署名ルート)
└───────┬────────┘
        ↓
┌────────────────┐
│   Tree Nodes   │
└───────┬────────┘
        ↓
┌────────────────┐
│     Record     │
└────────────────┘
</code></pre>

各ノードは[CID](https://github.com/multiformats/cid)ハッシュによって参照される[IPLD](https://ipld.io/)オブジェクト（[dag-cbor](https://ipld.io/docs/codecs/known/dag-cbor/)）です。上記の図での矢印はCIDの参照を表しています。

このレイアウトは[AT URIs](/specs/at-uri-scheme)に反映されています：

<pre><code>Root       | at://alice.com
Collection | at://alice.com/app.bsky.feed.post
Record     | at://alice.com/app.bsky.feed.post/1234
</code></pre>

データリポジトリへの「コミット」は単にRootノードのCIDに対するキーペア署名です。リポジトリへの各変更は新しいCommitノードを生成します。

## 識別子の種類

パーソナルデータリポジトリ内で複数の種類の識別子が使用されています。

<table>
  <tr>
   <td><strong>DID</strong>
   </td>
   <td><a href="https://w3c.github.io/did-core/">分散型識別子（DID）</a>はデータリポジトリを識別します。これは広くユーザーIDとして使用されますが、各ユーザーが1つのデータリポジトリを持つため、DIDはデータリポジトリへの参照と見なすことができます。使用される「DIDメソッド」によってDIDの形式は異なりますが、すべてのDIDは最終的にキーペアとサービスプロバイダのリストに解決されます。このキーペアはデータリポジトリへのコミットに署名できます。
   </td>
  </tr>
  <tr>
   <td><strong>CID</strong>
   </td>
   <td><a href="https://github.com/multiformats/cid">コンテンツID（CID）</a>はフィンガープリントハッシュを使用してコンテンツを識別します。これはリポジトリ全体でオブジェクト（ノード）を参照するために使用されます。リポジトリ内のノードが変更されると、そのCIDも変更されます。ノードを参照する親はその参照を更新し、これが親のCIDも変更することになります。これは最終的にCommitノードに連鎖し、それが署名されます。
   </td>
  </tr>
  <tr>
   <td><strong>NSID</strong>
   </td>
   <td><a href="/specs/nsid">ネームスペース識別子（NSID）</a>はリポジトリ内のレコードのグループのためのLexiconタイプを識別します。
   </td>
  </tr>
  <tr>
   <td><strong>rkey</strong>
   </td>
   <td><a href="/specs/record-key">レコードキー</a>（"rkeys"）は指定されたリポジトリ内のコレクション内の個々のレコードを識別します。フォーマットはコレクションLexiconによって指定され、一部のコレクションは "self" のようなキーを持つ単一のレコードのみを持ち、他のコレクションはタイムスタンプ識別子（TID）と呼ばれるbase32エンコードされたタイムスタンプを使用したキーを持っています。
   </td>
  </tr>
</table>

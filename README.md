# atproto-website-JA

このリポジトリは、ATプロトコルの公式ドキュメンテーションページを、ChatGPTを使用して日本語に自動翻訳したものです。
本リポジトリの誤訳等により生じたいかなる問題についても、当方は責任を負いかねます。

誤訳の修正はIssue、またはPull requestまでお願い致します。

以下は公式リポジトリのREADME.mdを翻訳したものです。

---

# atproto-website

このリポジトリには、ATプロトコルのドキュメンテーションが含まれています。[atproto.com](https://atproto.com/) で読むことができます。

Bluesky APIのドキュメンテーションを読むには、[docs.bsky.app](https://www.docs.bsky.app/) またはこの [リポジトリ](https://github.com/bluesky-social/bsky-docs) にアクセスしてください。
---

### atproto.comの編集

- このリポジトリをクローンします
- `npm install` を実行します
- 開発サーバーを `npm run dev` または `yarn dev` で実行します
- ブラウザで [http://localhost:3000](http://localhost:3000) を開きます。

---

[pages/index.js](https://github.com/bluesky-social/atproto-website/blob/main/pages/index.js) は [http://localhost:3000](http://localhost:3000) を生成します。変更を加えたい場合は、ここから始めてください。

ファイルを編集すると、ページが自動的に更新されます。

### atproto上で開発を行いたい開発者の方へ

BlueskyはATプロトコル上に構築されたオープンソーシャルネットワークで、開発者をエコシステムから締め出さない柔軟な技術で構築されています。atprotoを使用することで、サードパーティはカスタムフィード、連携サービス、クライアントなどを通じて、ファーストパーティと同じくらいスムーズに動作できます。

atproto上で開発を行いたい開発者の方は、Blueskyの招待コードをメールで送付させていただきます。GitHub（または類似のプロフィール）を[このフォーム](https://forms.gle/BF21oxVNZiDjDhXF9)からお知らせください。

## ライセンス

ドキュメンテーションテキストおよびatprotoの仕様はクリエイティブ・コモンズ・表示（CC-BY）のもとに提供されています。

インラインのコード例、例示データ、および正規表現はクリエイティブ・コモンズ・ゼロ（CC-0、別名パブリックドメイン）で提供され、帰属なしでコピー/貼り付けが可能です。

派生作品に関するリマインダーが記載されている [LICENSE.txt]() と、ライセンスの法的テキストのコピーがある [LICENSE-CC-BY.txt]() をご参照ください。
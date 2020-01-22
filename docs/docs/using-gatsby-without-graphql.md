---
title: GraphQLなしでGatsbyを使う
---

たいていの Gatsby に関するドキュメントや web 上の例では、データ取得プラグインをいかに活用してサイトのデータを管理するかというところにフォーカスしていると思います。 でもデータを Gatsby に取り込むためのデータ取得プラグイン（や Gatsby ノード）は、必ずしも必要ないのです！GraphQL を使わなくとも Gatsby だけで外部データを取り扱えます。

> 注意: ここでは、 “非構造データ”を “Gatsby における データ層の外で加工された”データのことを意味します (Gatsby のノードに変換せずに直接取得されたデータを使います)

## アプローチ: Gatsby の `createPages` API を使いデータを取得する

> _注意_: 下記サンプルコードは、 ここでいう”非構造化データ”アプローチをどのように取り扱うかを確かめるために作られたリポジトリです。 [View the full repo on GitHub](https://github.com/jlengstorf/gatsby-with-unstructured-data).

Gatsby プロジェクト内の `gatsby-node.js` ファイルに、 必要なデータ取得元を `createPages` API の中の`createPage`Action に記述してください。

```javascript:title=gatsby-node.js
exports.createPages = async ({ actions: { createPage } }) => {
  // `getPokemonData` はデータ取得のファンクションです
  const allPokemon = await getPokemonData(["pikachu", "charizard", "squirtle"])

  // 全ポケモンリストを表示するページを生成します
  createPage({
    path: `/`,
    component: require.resolve("./src/templates/all-pokemon.js"),
    context: { allPokemon }, // highlight-line
  })

  // 個別のポケモンを表示するページです
  allPokemon.forEach(pokemon => {
    createPage({
      path: `/pokemon/${pokemon.name}/`, // highlight-line
      component: require.resolve("./src/templates/pokemon.js"),
      context: { pokemon }, // highlight-line
    })
  })
}
```

- `createPages` は [Gatsby Node API]です。(/docs/node-apis/#createPages). [Gatsby の起動手順]の中で読み込まれます。(/docs/gatsby-lifecycle-apis/#bootstrap-sequence).
- [`createPage` アクション](/docs/actions/#createPage) 実際のページをものです。

ハイライトした行でデータは props としてアクセスできるページのテンプレートに埋め込まれます。

```jsx:title=/src/templates/pokemon.js
// highlight-next-line
export default ({ pageContext: { pokemon } }) => (
  <div style={{ width: 960, margin: "4rem auto" }}>
    {/* highlight-start */}
    <h1>{pokemon.name}</h1>
    <img src={pokemon.sprites.front_default} alt={pokemon.name} />
    {/* highlight-end */}
    <h2>Abilities</h2>
    <ul>
      {/* highlight-start */}
      {pokemon.abilities.map(ability => (
        <li key={ability.name}>
          <Link to={`./pokemon/${pokemon.name}/ability/${ability.name}`}>
            {ability.name}
            {/* highlight-end */}
          </Link>
        </li>
      ))}
    </ul>
    <Link to="/">Back to all Pokémon</Link>
  </div>
)
```

## どのような時に "非構造化データ"アプローチを用いるとよいでしょうか？

データ層を用いることがプロジェクトの規模に対して大げさと感じた時にこのアプローチを検討すると良いかもしれません。

## 非構造化データアプローチのよいところ

- 馴染みやすくとっつきやすい：GraphQL に慣れていない場合は特に、このアプローチは馴染みやすいかもしれません
- 余計な処理がない：取得したデータをストレートに渡し画面を生成します

## データ層を導入することで得られること

一方でデータ層を導入すると下記のようなメリットがあります。

- コンポーネント自体に必要なデータを宣言的に記述できます。
- フロントエンド側のデータ取得に関する同じようなコードを記述することを排除する。 — データの問い合わせと待ち時間を気にする必要がありません。 GraphQL クエリに必要なデータを問い合わせるだけで必要となった時に表示されます。
- フロントエンドの複雑な部分をクエリに追いやることができます。 — たいていのデータ加工は GraphQL クエリのビルド時に終えることができます。
- 階層の入り組んだ複雑なデータを扱うモダンなアプリケーションにとっては完璧なデータクエリ言語です。
- データの肥大化をなくすことでパフォーマンスを改善します。 — GraphQL によってビューで必要とされるデータを遅延読み込みしているため Gatsby は高速に動作します。
- 開発環境でのホットリロードを可能にします。 "ポケモン"の web サイトの例でいうと、「他のポケモンをみる」機能を詳細ページに追加したい時、`gatsby-node.js`はすべてのポケモンをページに読み込まなければなりません。さらに開発環境サーバーの再起動が必要です。 しかし GraphQL を利用するとクエリが追加できホットリロードされます。

> より深く [GraphQL](/docs/querying-with-graphql/)を知りたい時。

データ層の外側で処理をすることによって、下記リンクに示すようにトランスフォーマープラグインで提供されるような最適化が得られます。

- [`gatsby-image`](https://github.com/gatsbyjs/gatsby/tree/master/packages/gatsby-image) (speedy optimized images),
- [`gatsby-transformer-sharp`](https://github.com/gatsbyjs/gatsby/tree/master/packages/gatsby-transformer-sharp)（リサイズ、切り取り、レスポンシブな画像の生成といったいろいろな方法で画像処理をするためのクエリ可能なフィールドを提供します）
- Gatsby エコシステムのすべての公式のコミュニティーが作成した[トランスフォーマープラグイン](/plugins/?=transformer)など

もう 1 つの非構造データを用いた場合の難しさとして、複数のソースから直接データを取得する時にコードがますます発散してしまうことです。

## Gatsby にとっておすすめは？

もしあなたが作ろうと考えているサイトの規模が小さいものでしたら、サイトを生成するために効率の良い方法の 1 つはこのガイドのアウトラインのように`createPages` API を使って非構造データにひっぱりこむことです。それからもしサイトが後々複雑化していった時やより複雑なサイトの構築やデータを加工したくなった場合下記のような手順を実施ください。

1.  ほしいソース取得プラグインとトランスフォームプラグインが[Plugin Library](/plugins/)存在するかを確認ください。
2.  もしなければこちら[Plugin Authoring](/docs/creating-plugins/)をお読みいただきご自身で作ることをご検討ください！

## 参考

- Amberley Romo さんのガイド [using Gatsby without GraphQL](/blog/2018-10-25-using-gatsby-without-graphql/)
- [Why Gatsby Uses GraphQL](/docs/why-gatsby-uses-graphql/)

---
title: GraphQLを使わないGatsby
---

たいていの Gatsby に関するドキュメントや web 上の例では、Gatsby 製サイトのデータを管理するためにレバレッジのきいた（効率いい）プラグインにフォーカスしていると思います。 でも、Gatsby にデータを取り込むためにソースプラグイン（や Gatsby nodes）は、必ずしも必要ないのです！　また、 Gatsby 製のサイトの中の 非構造化データ アプローチを可能とします。GraphQL は必要ありません。

> 注意: ここでの目的として, “非構造データ”は “Gatsby データ層の外から操作された”データを意味します (直接データを操作し, and not transforming the data into Gatsby nodes)

## アプローチ: データを取得し Gatsby の `createPages` API を使う

> _注意_: この例では どのように非構造化データを利用するアプローチのモデルとして作られたサンプルリポジトリです。 [View the full repo on GitHub](https://github.com/jlengstorf/gatsby-with-unstructured-data).

Gatsby のプロジェクト内の `gatsby-node.js` ファイルの中に、 必要なデータを取得して `createPages` API の中の`createPage`Action にセットしてください。

```javascript:title=gatsby-node.js
exports.createPages = async ({ actions: { createPage } }) => {
  // `getPokemonData` is a function that fetches our data
  const allPokemon = await getPokemonData(["pikachu", "charizard", "squirtle"])

  // Create a page that lists all Pokémon.
  createPage({
    path: `/`,
    component: require.resolve("./src/templates/all-pokemon.js"),
    context: { allPokemon }, // highlight-line
  })

  // Create a page for each Pokémon.
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

ハイライトした行でデータは props としてアクセスできるところにページのテンプレートに埋め込まれます。

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

## どのような時に "非構造化データ" をつかうとよいでしょうか？

プロジェクトの規模に対して大きすぎると感じた時に Gatsby のデータ層を使うアプローチをとることは良いかもしれません。

## 非構造化データを使うといいところ

- The approach is familiar and comfortable, especially if you’re new to GraphQL
- There’s no intermediate step: you fetch some data, then build pages with it

## Gatsby のデータ層を推し進めることのトレードオフ

Gatsby のデータ層を使うこと下記のようなメリットがあります。

- どんなデータがコンポーネントに必要かを宣言的に特定することを可能にする。 alongside the page component
- フロントエンドのデータのボイラープレートを排除する。 — データのリクエスト待ちを気にする必要がない。 GraphQL クエリに必要なデータをきくだけで必要なとこに表示されます。
- フロントエンドの複雑な部分をクエリにまとめることができます。 — たいていのデータ加工は GraphQL クエリ野中の at build-time で終えることができます。
- 階層の入り組んだ複雑なデータに依存するようなモダンなアプリケーションにとって完璧なデータクエリ言語です。
- データ bloat をなくすことでパフォーマンスを改善します。 — GraphQL は Gatsby がそれぞれのビューで必要とされるデータを遅延ロード可能にすることによって Gatsby が速い理由です。
- 開発環境でのホットリローディングを可能にします。 "ポケモン"の web サイトの例でいうと、「他のポケモンをみる」機能を詳細ページに追加したい時、`gatsby-node.js`はすべてのポケモンをページに読み込まなければならない。さらに環境サーバのリスタートが必要です。 対してクエリを利用するとクエリが追加できホットロードされます。

> より深く [GraphQL](/docs/querying-with-graphql/)を知りたい時。

Working outside of the data layer also means foregoing the optimizations provided by transformer plugins, like:

- [`gatsby-image`](https://github.com/gatsbyjs/gatsby/tree/master/packages/gatsby-image) (speedy optimized images),
- [`gatsby-transformer-sharp`](https://github.com/gatsbyjs/gatsby/tree/master/packages/gatsby-transformer-sharp) (provides queryable fields for processing your images in a variety of ways including resizing, cropping, and creating responsive images),
- ... the whole Gatsby ecosystem of official and community-created [transformer plugins](/plugins/?=transformer).

Another difficulty added when working with unstructured data is that your data fetching code becomes increasingly hairy when you source directly from multiple locations.

## The Gatsby recommendation

If you're building a small site, one efficient way to build it is to pull in unstructured data as outlined in this guide, using `createPages` API, and then if the site becomes more complex later on, you move on to building more complex sites, or you'd like to transform your data, follow these steps:

1.  Check out the [Plugin Library](/plugins/) to see if the source plugins and/or transformer plugins you'd like to use already exist
2.  If they don't exist, read the [Plugin Authoring](/docs/creating-plugins/) guide and consider building your own!

## 参考

- Amberley Romo's guide to [using Gatsby without GraphQL](/blog/2018-10-25-using-gatsby-without-graphql/)
- [Why Gatsby Uses GraphQL](/docs/why-gatsby-uses-graphql/)

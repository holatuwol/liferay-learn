# GraphQL APIの使用

Liferay DXPには、ほとんどのアプリケーションに対応している[GraphQL](https://graphql.org) APIが含まれています。 ここでは、それらを使用する方法を学びます。 必要なステップはわずか3つです。

1.  使用するAPIを特定します。
2.  必要なデータを含むサイトを特定します。
3.  データにアクセスできる資格情報を使用してAPI呼び出しを行います。

この例では、Liferay DXPの新規インストールでDockerイメージを使用しています。

## 使用するサービスを特定する

GraphQL APIを呼び出すには、実行中のLiferay DXPが必要です。 Dockerを使用して取得するには、次のコマンドを実行します。

``` bash
docker run -it -p 8080:8080 liferay/portal:7.3.0-ga1
```

Liferay DXPの初期化後、必要なサービスを見つけることができます。 スキーマをリクエストすることで、ローカルインストールのAPIを検出できます。

``` bash
curl 'http://localhost:8080/o/graphql'  -H 'Content-Type: application/json' --data '{"query":"query{ __schema{ queryType{ name fields{ name args{ name } description } } } }","variables":{}}'
```

このURLは認証を必要としませんが、APIの呼び出しでは認証を必要とします。 返されるJSONは、利用可能なAPIを示しています。 この例では、`BlogPosting` APIを使用して[Blogs]ウィジェットからブログ投稿を取得していますが、この手順はリストされているどのAPIでも使用できます。

``` tip::
   上記の``curl``コマンドを実行すると、返されるスキーマを管理するのが非常に面倒であることがわかります。 このため、APIをわかりやすく読みやすい形式で表示できるGraphQLクライアントを使用することをお勧めします。 クライアントのリストは、`awesome-graphql <https://github.com/chentsulin/awesome-graphql#tools>`_ GitHubリポジトリにあります。 以下の例の残りは、`Insomnia <https://insomnia.rest/graphql/>`_クライアントを使用して作成されました。
```

`BlogPosting` APIは次のようになります。

``` graphql
createSiteBlogPosting(
  blogPosting: InputBlogPosting
  siteKey: String!
  ): BlogPosting
```

APIでは、エントリが投稿されるブログを含むサイトを把握している必要があるため、最初にサイトIDを見つける必要があります。

## データを含むサイトを特定する

Liferay DXPの初期化後、ブラウザで`http://localhost:8080`にアクセスします。

ここで、デフォルトのサイトIDを見つける必要があります。

1.  デフォルトの認証情報を使用してサインインします（**ユーザー名：** `test@liferay.com` **パスワード：** `test`）。
2.  [Control Panel] → [Sites] → [Sites]に移動します。
3.  [Actions]ボタンをクリックし、*[Go to Site Settings]*を選択します。

サイトIDが[Details]セクションの上部に表示されます。 `20119`のような整数になります。

## データにアクセスできる認証情報を使用してサービス呼び出しを行う

これで、呼び出しを行うために必要なものがすべて揃いました。 すべてのWebサービスには、要求しているデータにアクセスできる資格情報を使用してアクセスする必要があります。 最も簡単な方法は、URLで資格情報データを渡す基本認証を使用することです。 これは安全ではないため、この方法は開発時にのみ使用すべきです。 本番環境では、アプリケーションは[OAuth2](../../installation-and-upgrades/securing-liferay/configuring-sso/using-oauth2/introduction-to-using-oauth2.md)を介してユーザーを承認する必要があります。

### 基本認証を使用したGraphQL APIの呼び出し（開発中のみ）

基本認証を使用してサービスを呼び出すには、URLに資格情報を指定します。

``` bash
curl --request POST --url http://localhost:8080/o/graphql \ -u test@liferay.com:test  --header 'content-type: application/json' --data '{"query":"query {blogPostings(filter: \"\", page: 1, pageSize: 10, search: \"\", siteKey: \"20119\", sort: \"\"){ page  items{ id articleBody headline  creator{ name }}}}"}'
```

### OAuth2を使用してサービスを呼び出す

本番環境では、[OAuth2アプリケーション](../../installation-and-upgrades/securing-liferay/configuring-sso/using-oauth2/creating-oauth2-applications.md)を作成し、OAuth2プロセスを使用して認証トークンを取得します。 トークンを取得したら、それをHTTPヘッダーに指定します。

``` bash
curl --request POST --url http://localhost:8080/o/graphql -H "Authorization: Bearer d5571ff781dc555415c478872f0755c773fa159" --header 'content-type: application/json' --data '{"query":"query {blogPostings(filter: \"\", page: 1, pageSize: 10, search: \"\", siteKey: \"20119\", sort: \"\"){ page  items{ id articleBody headline  creator{ name }}}}"}'
```

## データの取得と投稿

上記のクエリを実行してすべてのブログ投稿を取得すると、何もないことがわかります。

``` json
{"data":{"blogPostings":{"page":1,"items":[]}}}
```

まず、ブログエントリを投稿します。

### ブログエントリの投稿

GraphQLスキーマによって、ブログエントリを投稿するために行う必要がある呼び出しが明らかになりました。

1.  公開するエントリを含むJSONドキュメントを作成します。

    ``` json
    {
      "blog": {
          "articleBody": "This Blog entry was created by calling the GraphQL service!",
          "headline": "GraphQL Blog Entry"
      }
    }
    ```

2.  スキーマドキュメンテーションに基づいてGraphQLクエリを作成します。

    ``` graphql
    mutation CreateBlog($blog: InputBlogPosting){
      createSiteBlogPosting(blogPosting: $blog, siteKey: "20119" ) {
        headline
        articleBody
        id
        friendlyUrlPath
      }

     }
    ```

3.  リクエストを行います。

    ``` bash
    curl --request POST --url http://localhost:8080/o/graphql -u test@liferay.com:test --header 'content-type: application/json' --data '{"query":"mutation CreateBlog($blog: InputBlogPosting){   createSiteBlogPosting(blogPosting: $blog, siteKey: \"20119\" ) {    headline    articleBody    id    friendlyUrlPath  }    } ","variables":{"blog":{"articleBody":"This Blog entry was created by using Curl to call the GraphQL service!","headline":"Curl GraphQL Blog Entry"}},"operationName":"CreateBlog"}'
    ```

Liferay DXPは、ミューテーションでリクエストされたフィールドを含むブログエントリのJSON表現を返します。

``` json
{
  "data": {
    "createSiteBlogPosting": {
      "headline": "GraphQL Blog Entry",
      "articleBody": "This Blog entry was created by calling the GraphQL service!",
      "id": 35512,
      "friendlyUrlPath": "graphql-blog-entry"
    }
  }
}
```

GraphQLクライアントは、スキーマを自動解析してコード補完を提供できるため、APIを呼び出す作業が簡単になります。

![InsomniaのようなGraphQLクライアントは、スキーマを解析してコードを完成させることにより、クエリとミューテーションを構築するのに役立ちます。](./consuming-graphql-apis/images/01.png)

### すべてのブログエントリを取得する

最初に行った呼び出しによって次のAPIが呼び出されました。

``` graphql
blogPostings (
   filter:String
   page: Int
   pageSize: Int
   search: String!
   sort: String
): BlogPostingPage
```

ここで、最初のクエリを繰り返して、投稿したブログエントリが表示されることを確認できます。

``` bash
curl --request POST --url http://localhost:8080/o/graphql \ -u test@liferay.com:test  --header 'content-type: application/json' --data '{"query":"query {blogPostings(filter: \"\", page: 1, pageSize: 10, search: \"\", siteKey: \"20119\", sort: \"\"){ page  items{ id articleBody headline  creator{ name }}}}"}'
```

ここで、Liferay DXPは、要求されたデータを含むJSONを返します。

``` json
{
  "data": {
    "blogPostings": {
      "page": 1,
      "items": [
        {
          "id": 35512,
          "articleBody": "This Blog entry was created by calling the GraphQL service!",
          "headline": "GraphQL Blog Entry",
          "creator": {
            "name": "Test Test"
          }
        }
      ]
    }
  }
}
```

### 単一のブログエントリを取得する

単一のブログエントリを取得するためのGraphQLスキーマからのAPI呼び出しには、パラメーターが1つしかありません。

``` graphql
blogPosting(
   blogPostingId: Long
): BlogPosting
```

上記のクエリでブログ投稿のIDが判明したため、必要な投稿のみを取得できます。

``` bash
curl --request POST --url http://localhost:8080/o/graphql -u test@liferay.com:test --header 'content-type: application/json' --data '{"query":"query {blogPosting(blogPostingId: 35512){ id  headline  articleBody}}"}'
```

同じブログエントリが返されます。

``` json
{
  "data": {
    "blogPosting": {
      "id": 35512,
      "headline": "GraphQL Blog Entry",
      "articleBody": "This Blog entry was created by calling the GraphQL service!"
    }
  }
}
```

### ブログエントリの削除

ブログエントリの削除は、作成と同様にミューテーションです。 その呼び出しは、単一のブログエントリを取得することとほぼ同じです。

``` graphql
deleteBlogPosting(
  blogPostingId: Long
): Boolean
```

Curlを使用すると、次のように呼び出すことができます。

``` bash
curl --request POST --url http://localhost:8080/o/graphql -u test@liferay.com:test --header 'content-type: application/json' --data '{"query":"mutation {deleteBlogPosting(blogPostingId: 35512)}"}'
```

この呼び出しは、成功または失敗を示すブール値をJSONドキュメントで返します。

``` json
{
  "data": {
    "deleteBlogPosting": true
  }
}
```

　 Liferay DXPのGraphQLサービスを呼び出す方法を学びました。 上記の例では基本認証を使用していることに注意してください。本番環境では、OAuth2を使用して安全な方法でサービスを呼び出します。

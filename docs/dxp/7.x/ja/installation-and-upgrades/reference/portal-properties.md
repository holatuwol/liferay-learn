# ポータルプロパティ

DXPを初期状態で実行するために必要なすべての設定は、*ポータルプロパティ*を使用して指定します。 プロパティは、サーバーの起動時にDXPがプロパティファイルから読み取る名前と値のペアのセットです。 プロパティのデフォルトは、DXPインストールの`portal-impl.jar/portal.properties`ファイルに指定されています。 [ポータルプロパティ](https://docs.liferay.com/dxp/portal/7.2-latest/propertiesdoc/portal.properties.html)リファレンスには、すべてのプロパティがリストされ、説明、値の例、初期値が含まれています。

アプリケーションサーバーの起動が完了すると、一部のプロパティはユーザーインターフェイス（UI）を介して変更できますが、サーバーを起動する前にプロパティファイルで変更する必要があるプロパティもあります。 プロパティファイルを介して行わなければ*ならない*設定の例には、データベースへの接続、[`[Liferay Home]`](./liferay-home.md)フォルダの場所の宣言、ユーザー認証方法の変更（電子メールアドレスではなく画面名による）、およびファイルアップロードのサイズ制限の増加などがありますが、これらに限定されません。

``` warning::
   portal-impl.jar/portal.propertiesファイルを直接変更しないでください。代わりに、別のプロパティファイル（拡張ファイル）を使用して、変更するプロパティをオーバーライドします。 作成する一般的な拡張ファイルは、`[Liferay Home] <./liferay-home.md>`_または[USER_HOME]フォルダでportal-ext.propertiesと呼ばれます。
```

ポータルプロパティを使用してDXPインストールを構成することは、Liferay DXPを構成する最も一般的で推奨される方法であり、次の利点も提供されます。

  - 他のLiferay DXP環境およびサーバーノードに簡単に配布できる単一の構成ファイル。
  - 構成管理を簡素化するために、構成をバージョン管理システムにチェックインする機能。
  - 最初の起動前にすべてのカスタムプロパティをファイルに設定することがDXPを構成する最速の方法です。

## ポータルプロパティの使用

`[Liferay Home]/portal-ext.properties`は、使用する最も一般的な拡張ファイルです。 `portal-ext.properties`ファイルがなく、セットアップウィザードを使用して変更を適用する場合、DXPはこれらのプロパティを`portal-setup-wizard.properties`というファイルに設定します。

以下は、`portal.properties`ファイルで設定できる構成の例です。

### データベース接続の設定

データベース接続プロパティは通常、`portal-ext.properties`ファイルに設定されます。 たとえば、データベース接続を変更する場合は、`portal-ext.properties`ファイルを作成し、[データベース接続プロパティ](./database-templates.md)を必要な値に設定します。

``` properties
jdbc.default.driverClassName=org.mariadb.jdbc.Driver
jdbc.default.url=jdbc:mariadb://localhost/lportal?useUnicode=true&characterEncoding=UTF-8&useFastDateParsing=false
jdbc.default.username=joe.bloggs
jdbc.default.password=123456
```

データベース構成の詳細については、[Database Configurations](./database-configurations.md)および[Database Templates](./database-templates.md)を参照してください 。

### Liferay Homeの場所の設定

一部のアプリケーションサーバー（WebLogicなど）では、DXP WARファイルをデプロイする前に[Liferay Home](https://help.liferay.com/hc/en-us/articles/360028831932-Installing-Liferay-DXP-on-WebLogic-12c-R2)の場所をカスタマイズする必要があります。 [`liferay.home`](https://docs.liferay.com/dxp/portal/7.2-latest/propertiesdoc/portal.properties.html#Liferay%20Home)プロパティを使用すると、場所を設定できます。

``` properties
liferay.home=/home/jbloggs/liferay
```

### ユーザーの認証方法の変更

ユーザーがLiferay DXPサーバーに認証される方法を変更するには、`portal-ext.properties`ファイルに以下を追加します。

``` properties
company.security.auth.type=emailAddress
#company.security.auth.type=screenName
#company.security.auth.type=userId
```

## 構成の優先度

Liferayの構成には微妙な差異があり、一部の構成を複数の場所（複数のポータルプロパティファイルまたはUI）に設定できます。

### ポータルプロパティの読み込み順序

一度に複数のポータルプロパティのオーバーライドファイルを作成してデプロイすることが可能です。 この状況では、Liferay DXPは、プロパティ値の読み取りと適用の順序に優先順位を付けます。`1.`の優先順位が一番高くなります。

1.  `${Liferay Home}/portal-setup-wizard.properties`
2.  `${USER_HOME}/portal-setup-wizard.properties`
3.  `${Liferay Home}/portal-ext.properties`
4.  `${USER_HOME}/portal-ext.properties`
5.  `${Liferay Home}/portal-bundle.properties`
6.  `${USER_HOME}/portal-bundle.properties`
7.  `${Liferay Home}/portal.properties`
8.  `portal-impl.jar/portal.properties`

この順序自体は、DXPの[`include-and-override`](https://docs.liferay.com/dxp/portal/7.2-latest/propertiesdoc/portal.properties.html#Properties%20Override) ポータルプロパティを設定することで変更および構成できます。

``` note::
   DXPインストールの構成管理を簡略化するために、必要な数のプロパティファイルを使用することをお勧めします。
```

### UI構成とポータルプロパティ

一部のポータルプロパティは、[システム設定](https://help.liferay.com/hc/en-us/articles/360029131591-System-Settings)として、*[Control Panel]* → *[Configuration]* → *[System Settings]*または`.config`ファイルで変更できます。 これらのプロパティは、DXPデータベースに格納されます。 たとえば、SAML認証プロパティは、システム設定で利用可能なポータルプロパティです。

``` important::
   UIで設定されたプロパティは、ポータルプロパティファイルで設定されたプロパティよりも優先されます。
```

## 追加情報

  - [7.2 Portal Properties](https://docs.liferay.com/dxp/portal/7.2-latest/propertiesdoc/portal.properties.html)
  - [System Settings](https://help.liferay.com/hc/en-us/articles/360029131591-System-Settings)

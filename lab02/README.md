Lab 2 : APIを作成する
===

HTTP API エンドポイントを作成するには、まずCamelコンテキストにServletを追加する必要があります。 **Camel Contexts** の下にある **camel-context.xml** ファイルを開き、*source* タブを表示し、下記のコードを `<camelContext..>` タグの前に追加します。

```
    ...
    <bean class="org.apache.camel.component.servlet.CamelHttpTransportServlet" id="camelHttpTransportServlet"/>
    <bean
        class="org.springframework.boot.web.servlet.ServletRegistrationBean" id="servlet">
        <property name="name" value="CamelServlet"/>
        <property name="servlet" ref="camelHttpTransportServlet"/>
        <property name="urlMappings" value="/myfuselab/*"/>
    </bean>
    ...
```

同じファイル内で、 `<camelcontext..>` タグの下に、下記コードを追加し、RESTエンドポイントを設定します。

```
    ...
       <restConfiguration apiContextPath="api-docs" bindingMode="json"
            component="servlet" contextPath="/myfuselab">
            <apiProperty key="cors" value="true"/>
            <apiProperty key="api.title" value="My First Camel API Lab"/>
            <apiProperty key="api.version" value="1.0.0"/>
        </restConfiguration>
	<!-- Right above route id="customer" -->    
	...
```

次にAPIエンドポイントを１つ作成するため、 **restConfiguration** の下に下記コードを追加します。

```
    ...
        <rest path="/customer">
            <get uri="all">
            	<description>Retrieve all customer data</description>
                <to uri="direct:getallcustomer"/>
            </get>
        </rest>
    ...
```

さらに、タイマーでデータベースの検索をトリガーする代わりに、作成したAPIコールによりトリガーするように変更します。Camelルートにある **Timer** を **Direct** コンポーネントに置き換えます。

下記の記述を、

```
<from id="time1" uri="timer:timerName?repeatCount=1"/>
```

次のように変更します。

```
<from id="direct1" uri="direct:getallcustomer"/>
```

続いて **pom.xml** ファイルに依存するライブラリを追加します。

```
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.camel</groupId>
      <artifactId>camel-servlet-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.camel</groupId>
      <artifactId>camel-jackson-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.camel</groupId>
      <artifactId>camel-swagger-java-starter</artifactId>
    </dependency>
```

Project Explorerパネルの **myfuselab** を右クリックし、**Run As..** -> **Maven build** を選択してCamelアプリケーションを開始します。コマンドラインコンソールを開いて以下のコマンドを実行してみましょう。

```
curl -i http://localhost:8080/myfuselab/customer/all
```

顧客データのリストがJSONフォーマットで返ってくることを確認します。

```
[{"CUSTOMERID":"A01","VIPSTATUS":"Diamond","BALANCE":1000},{"CUSTOMERID":"A02","VIPSTATUS":"Gold","BALANCE":500}]
```

オプション: アプリケーションを停止して、別のAPIエンドポイントを作成してみましょう。Customer IDを引数として、IDにマッチする顧客データを返すGETメソッドを追加します。

RESTサービスの仕様はSwaggerドキュメントとして生成されます。Swagger ドキュメントを表示するためには、以下のコマンドを実行します。

```
curl -i http://localhost:8080/myfuselab/api-docs
```

#### ヒント!

* 新しいRESTエンドポイントのGETメソッドでuriを以下のように定義することで、custidを引数に持つことができます。
	* uri="{custid}"
* SQLコンポーネントで customerid をパラメータとして受け付けるSQL文を記述します。
	* select * from customerdemo where customerID=:#custid

Swaggerドキュメントを確認し、APIをテストします。顧客A01のデータが正しくJSONフォーマットで返ってくるか確認します。

```
curl -i http://localhost:8080/myfuselab/api-docs
curl -i http://localhost:8080/myfuselab/customer/A01
```

```
[{"CUSTOMERID":"A01","VIPSTATUS":"Diamond","BALANCE":1000}]
```

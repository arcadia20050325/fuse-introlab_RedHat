## Lab 1 : 初めてのSprint Bootプロジェクトを作る
JBoss Developer StudioのProject Explorerパネルで右クリックし、**New** -> **Fuse Integration Project** を選んで新しいプロジェクトを作成します。

![01-FIS-project.png](./img/01-FIS-project.png)

プロジェクト名に **myfuselab** を入力し、*next* をクリックします。

重要 : Camelバージョンに **2.18.1.redhat-000012** を選んでください。

*next* をクリックします。

![02-runtime.png](./img/02-runtime.png)

Advance project setup画面で、 **Use a predefined template** を選択し、**Fuse on OpenShift** -> **SprintBoot on OpenShift** を選択して *finish* をクリックします。

![03-template.png](./img/03-template.png)

Fuse perspectiveに変更するかと聞いてきますので、yesをクリックします。

*src/main/resources* ディレクトリにある **application.properties** をカット＆ペーストで複製し、**application-dev.properties** という名前で保存します。これを開発用の設定情報として利用します。

![04-devproperties.png](./img/04-devproperties.png)

**application-dev.properties** のデータソース構成として以下を追加します。

```
#Database configuration
spring.datasource.url = jdbc:h2:mem:mydb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
spring.datasource.username = sa
spring.datasource.password = 
spring.datasource.driver-class-name = org.h2.Driver
spring.datasource.platform = h2
```
*注意: 今回はテスト用として、H2のインメモリデータベースを利用します。 SpringBootのAutoConfiguration機能により、Camel contextのデフォルトデータソースとして自動的にロードされます。*

*src/main/resources* フォルダで右クリックして、**New** -> **Others**を選択して新規ファイルを作成します。

![05-newfile.png](./img/05-newfile.png)

ウィザードで **File** を選択し、nextをクリックします。

ファイル名に **schema.sql** を指定します。myfuselabプロジェクトの *src/main/resources* に作成されることを確認してfinishを選択します。

![06-schemasql.png](./img/06-schemasql.png)

以下のSQL文を **schema.sql** に追加します。

```
DROP TABLE IF EXISTS `customerdemo`;
CREATE TABLE customerdemo (
	customerID varchar(10) NOT NULL,
	vipStatus varchar(10) NOT NULL ,
	balance integer NOT NULL
);

INSERT INTO customerdemo (customerID,vipStatus,balance) VALUES ('A01','Diamond',1000);
INSERT INTO customerdemo (customerID,vipStatus,balance) VALUES ('A02','Gold',500);
```

![07-sql.png](./img/07-sql.png)

**Camel Contexts** の下にある **camel-context.xml** ファイルをダブククリックすると、Camelルートが確認できます。
今回新規に作成するのでキャンバスに表示されるデフォルトのルートを削除します。

![08-deleteroute.png](./img/08-deleteroute.png)

右側にある *Routing* パレットから、**ROUTE** コンポーネントをドラッグして新しいルートを作成します。プロパティセクションにある *ID* テキストボックスにルート名 **customer** を設定します。

![09-route.png](./img/09-route.png)

ルートを起動するために Timer を利用します。右側にある *Components* パレットから **TIMER** コンポーネントをドラッグし、キャンバス内のルートにドロップします。 *Properties*-> *Advance* タブ -> *Consumer* を表示し、 **Repeat Count** プロパティに **1**　を入力します。

![10-timer.png](./img/10-timer.png)

データソースからデータを読み込むため、 *Components* パレットから **SQL** コンポーネントを選択します。 *Properties*-> *Advance* タブ -> *Path* を表示して、 **Query** プロパティに **select * from customerdemo** を設定します。 

![11-sqlcomponent.png](./img/11-sqlcomponent.png)

さらに *Common* タブで、 **Data Source** プロパティに **dataSource** を設定します。

![12-datasource.png](./img/12-datasource.png)

最後に **LOG** コンポーネントを選択し、*Components* パレットからルートに持って来ます。 *Properties*-> *Detail* タブを表示し、 **Message** プロパティに **${body}** を設定します。

![13-log.png](./img/13-log.png)

アプリケーションを開始する前に、データベースドライバーの依存性を **pom.xml** に追加します。また作成したapplications-dev.propertiesを利用するため、プロファイルにdevを指定します。

```
...
<properties>
  ...
  <run.profiles>dev</run.profiles>
</properties>
...

<dependencies>
	...
    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
    </dependency>
    ...
</dependencies>
```
Project Explorerパネルの **myfuselab** を右クリックし、**Run As..** -> **Maven build...** を選択します。

![14-mavenrun.png](./img/14-mavenrun.png)

ポップアップウィンドウで、 *Goals* プロパティに **spring-boot:run** を入力し、 **Skip Tests** プロパティをチェックします。

![15-springbootrun.png](./img/15-springbootrun.png)

*Run*ボタンをクリックして実行し、ログコンソールに顧客データが表示されていることを確認します。
```
customer - [{CUSTOMERID=A01, VIPSTATUS=Diamond, BALANCE=1000}, {CUSTOMERID=A02, VIPSTATUS=Gold, BALANCE=500}]
```

## Lab 3 - OpenShiftにデプロイする

###CDKを起動する

まだOpenShiftを起動していない場合は、コマンドプロンプトを開いて以下のコマンドを実行します。

```
oc-cluster up your_name
```

以下の応答が返ってきたら、OpenShiftの起動が成功です。

```
$ oc-cluster up yourname
# Using client for ocp v3.5.5.31
[INFO] Created self signed certs. You can avoid self signed certificates warnings by trusting this certificate: /home/ec2-user/.oc/certs/master.server.crt
[INFO] Running a previously created cluster
oc cluster up --version v3.5.5.31 --image registry.access.redhat.com/openshift3/ose --public-hostname 127.0.0.1 --routing-suffix apps.127.0.0.1.nip.io --host-data-dir /home/ec2-user/.oc/profiles/test/data --host-config-dir /home/ec2-user/.oc/profiles/test/config --host-pv-dir /home/ec2-user/.oc/profiles/test/pv --use-existing-config -e TZ=EDT
-- Checking OpenShift client ... OK
-- Checking Docker client ... OK
-- Checking Docker version ... OK
-- Checking for existing OpenShift container ... OK
-- Checking for registry.access.redhat.com/openshift3/ose:v3.5.5.31 image ... OK
-- Checking Docker daemon configuration ... OK
-- Checking for available ports ... 
   WARNING: Binding DNS on port 8053 instead of 53, which may not be resolvable from all clients.
-- Checking type of volume mount ... 
   Using nsenter mounter for OpenShift volumes
-- Creating host directories ... OK
-- Finding server IP ... 
   Using 172.31.28.24 as the server IP
-- Starting OpenShift container ... 
   Starting OpenShift using container 'origin'
   Waiting for API server to start listening
   OpenShift server started
-- Removing temporary directory ... OK
-- Checking container networking ... OK
-- Server Information ... 
   OpenShift server started.
   The server is accessible via web console at:
       https://127.0.0.1:8443

   To login as administrator:
       oc login -u system:admin

-- Permissions on profile dir fixed
Switched to context "test".
```

ブラウザから https://127.0.0.1:8443/console にアクセスして、OpenShiftコンソールを表示します。 
![00-openshift.png](./img/00-openshift.png)


OpenShiftにアプリケーションをデプロイする前に、これまでインメモリのH2データベースでテストして来ましたので、これを正規のデータベースで実行できるようにします。以下のデータソース設定を、 *src/main/resources* フォルダにある **application.properties** ファイルに追加します。

```
#mysql specific
mysql.service.name=mysql
mysql.service.database=sampledb
mysql.service.username=dbuser
mysql.service.password=password

#Database configuration
spring.datasource.url = jdbc:mysql://${${mysql.service.name}.service.host}:${${mysql.service.name}.service.port}/${mysql.service.database}
spring.datasource.username = ${mysql.service.username}
spring.datasource.password = ${mysql.service.password}
```

今回はMYSQL データベースを使うため、**pom.xml** にドライバーの依存性を追加します。

```
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <scope>runtime</scope>
</dependency>
```


トップメニューから window -> Show view -> others を選択します。ポップアップウィンドウでopenshiftを検索し、OpenShift Explorerを選択します。
![00-view.png](./img/00-view.png)
![00-openshiftexplorer02.png](./img/00-openshiftexplorer.png)

OpenShift Explorerの現在接続中のOpenShiftの接続を右クリックして、**NEW** -> **Project** から新規プロジェクトを作成します。
![01-newproject.png](./img/01-newproject.png)

プロジェクト名を **myfuseproject** 、表示名を **My Fuse Project** とします。

![02-projectname.png](./img/02-projectname.png)

In side the project we are going to first create a MYSQL database for our appkication, right click on the new project name **myfuseproject** -> **New** -> **Application**

![03-newapp.png](./img/03-newapp.png)

Server application sourceのところで **mysql-ephemeral(database, mysql) - openshift** を選択してNextをクリックします。

![04-mysql.png](./img/04-mysql.png)

以下のパラメータを設定します。

```
MYSQL_PASSWORD = password
MYSQL_USER = dbuser
```
![05-param.png](./img/05-param.png)

Finishをクリックすると、OpenShift Explorerにmysqlインスタンスが実行されていることが確認できます。

![06-mysqlcreated.png](./img/06-mysqlcreated.png)

Now we can finally push our application to OpenShift by right click on your project in project explorer. Select **Run As** -> **Run Configurations...**

![07-runmvn.png](./img/07-runmvn.png)

In the pop-up menu, select **Deploy myfuselab on OpenShift** on the left panel. Go to  **JRE** tab on the right, inside VM arguments, update kuberenets.namespace to **myfuseproject** and username/password to **openshif-dev/devel**. And **RUN**.

![08-runconfig.png](./img/08-runconfig.png)

To see everything running, in your browser, go to *https://<OPENSHIFT_IP>:8443/console* and login with **<ID>/<password>** (for people using *oc cluster up or wrapper, it's developler/developer*). Select **My Fuse Project**. And you will see both application in the overview page.

![09-overview.png](./img/09-overview.png)

OpenShiftの外のサービスにアクセスするため、左のメニューから**Application** -> **Service** を選択し、サービスページにある **camel-ose-springboot-xml** を選択します。

![10-service.png](./img/10-service.png)

**Create route**　をクリックします。

![11-createroute.png](./img/11-createroute.png)

そのままCreateをクリックします。

以下のURLでAPIエンドポイントにアクセスします。

```
curl http://<YOUR_ROUTE>/myfuselab/customer/all
curl  http://<YOUR_ROUTE>/myfuselab/customer/A01
```

顧客データがJSONフォーマットで返ってくることを確認します。
```
[{"CUSTOMERID":"A01","VIPSTATUS":"Diamond","BALANCE":1000},{"CUSTOMERID":"A02","VIPSTATUS":"Gold","BALANCE":500}]

[{"CUSTOMERID":"A01","VIPSTATUS":"Diamond","BALANCE":1000}]
```
Camelルートの実行状況を確認するためには、OpenShift コンソールから**Application** -> **pod** に行き、先頭の **camel-ose-springboot-xml-1-xxxxx** ポッドを選択します。

![12-podlist.png](./img/12-podlist.png)

**Open Java Console** をクリックし、Camelルートの状態を確認することができます。

![13-pod.png](./img/13-pod.png)

**Route Diagram** をクリックし、実行中の状況を確認することができます。

![14-javaconsole.png](./img/14-javaconsole.png)

下記のようにMySQLデータベースにログインして、データベースの内容を確認することもできます。

```
oc project myfuseproject

oc get pods
NAME                                   READY     STATUS    RESTARTS   AGE
camel-ose-springboot-xml-s2i-1-build   1/1       Running   0          15s
mysql-1-xxxxx                          1/1       Running   0          2m

oc rsh mysql-1-xxxxx

sh-4.2$ mysql -udbuser -p sampledb
Enter password: 

mysql> select * from customerdemo;
+------------+-----------+---------+
| customerID | vipStatus | balance |
+------------+-----------+---------+
| A01        | Diamond   |    1000 |
| A02        | Gold      |     500 |
+------------+-----------+---------+
2 rows in set (0.00 sec)
```
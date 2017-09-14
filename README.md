# Agile Integration Workshop
(Red Hat Fuse 6.3 Workshop - Fuse Integration Service 2.0)

これはOpenShift上で動作するFuse SpringBootプロジェクトを作成するシンプルなハンズオンチュートリアルです。
このチュートリアルは以下の4つのセクションで構成されています。

* データベースからデータを読み込むプロジェクトを作成する
* RESTful APIエンドポイントを公開する
* 作成したアプリケーションをOpenShiftにデプロイする
* 3scaleを使ってAPIを管理・制御する

## Red Hat 3scale API管理プラットフォームのアカウント登録
このチュートリアルではRed Hat 3scaleのデプロイと管理にフォーカスします。
ここで紹介するデプロイパターンはハイブリッドアプローチというものです。ハイブリッドアプローチは3scaleのAPIゲートウェイを自分の環境に設定する方法です。
このゲートウェイはパブリッククラウドで管理された、Red Hat 3Scale SaaS API管理プラットフォーム (以下AMP）と双方向通信を行います。

![00-3scale.png](./img/00-3scale.png)

lab4のチュートリアルでこのハイブリッドアプローチを使いますので、事前にRed Hat 3Scale SaaS AMPの無料トライアルアカウントを登録してください。（https://www.3scale.net/signup）

登録したメールアドレスにメールが届きますので、アカウントのアクティベーションを行い登録プロセスを完了します。

## インストール
以下のソフトウェアが事前にインストールされていることを確認してください。

* JBoss Development Suite V1.3 (MacOSX/Windows)
	* JBoss Developer Studio 10.3.0.GA (Integration SOA プラグインがインストールされていること）
	https://developers.redhat.com/products/devsuite/download/
	* Java Platform, Standard Edition 1.8.0.111
	* Red Hat Container Development Kit 2.4.0.GA
	* Oracle Virtualbox 5.0.26
	* Vagrant 1.8.1

## 開発環境のインストールと設定
JBoss Development Suiteをダブルクリックし、あなたが登録したRed Hat Developerサイトの認証情報でログインします。

![01-login.png](./img/01-login.png)

インストールフォルダの宛先を指定します。
インストーラーで指定したバージョンのコンポーネントを選択し、コンポーネントのダウンロードとインストールが開始されます。

![02-components.png](./img/02-components.png)

インストールが終わると、プロジェクトのワークスペースを選択する画面が表示されます。任意の場所を指定してください。

Red Hat JBoss Developer Studioの画面の真ん中のパネルにある "Software/Update"タグを選択します。"JBoss Fuse Development"ボックスをクリックし、Install/Update ボタンをクリックします。

![04-plugin.png](./img/04-plugin.png)

Red Hat JBoss Developer Studioが再起動されます。

## コンテナ開発キット（Container Development Kit）をインストールし設定する

Development Suiteをインストールしたフォルダの直下に、 **cdk** という名前のフォルダがあります。**${DEVSUITE_INSTALLTION_PATH}/cdk/components/rhel/rhel-ose/** に移動して、 **Vagrantfile** を編集します。

IMAGE_TAG を探して、OCPバージョンを v3.4 に設定してファイルを保存します。

```
#Modify IMAGE_TAG if you need a new OCP version e.g. IMAGE_TAG="v3.3.1.3"
IMAGE_TAG="v3.4"
```

コマンドラインコンソールでローカルのOpenShiftを起動します。

```
vagrant up
```

oc バイナリクライアントをインストールして設定します。

```
vagrant service-manager install-cli openshift
export PATH=${vagrant_dir}/data/service-manager/bin/openshift/1.4.0:$PATH
eval "$(VAGRANT_NO_COLOR=1 vagrant service-manager install-cli openshift  | tr -d '\r')"
```

admin でログインします。

```
oc login -u admin
Authentication required for https://10.1.2.2:8443 (openshift)
Username: admin
Password:
Login successful.

```

チュートリアルで使用する、Fuseのイメージストリームとデータベースのテンプレートをインストールします。

```
#FIS image
oc create -f https://raw.githubusercontent.com/jboss-fuse/application-templates/master/fis-image-streams.json -n openshift

#MYSQL Database
oc create -f https://raw.githubusercontent.com/openshift/origin/master/examples/db-templates/mysql-ephemeral-template.json -n openshift
```

developer でログインします。

```
oc login -u openshift-dev
Authentication required for https://10.1.2.2:8443 (openshift)
Username: openshift-dev
Password:
Login successful.

```

以下のURLから Access OpenShift コンソールにアクセスします。

```
https://10.1.2.2:8443
```

Red Hat JBoss Developer Studioにビュー戻り、OpenShift Explorer ビューで **New Connection Wizard..** をクリックして OpenShiftを設定します。
**Server** に **https://10.1.2.2:8443** を入力し、**retrieve** リンクをクリックしてトークンを入手します。

![05-token.png](./img/05-token.png)

ポップアップウィンドウでdeveloper/developerでログインします。OKを選択して **Save token** ボックスをチェックします。

![06-connection.png](./img/06-connection.png)

## Windows利用の場合

- コントロールパネルで Hyper-V 機能を無効にします。
- Add _config.ssh.insert\_key=false_ to **Vagrantfile** ${DEVSUITE_INSTALLATION_PATH}/cdk/components/rhel/rhel-ose/

Thanks to @sigreen


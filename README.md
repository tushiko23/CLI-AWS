## AWS CLIを使用して、環境を構築する

* AWS CLIを使用したAWSリソースの環境構築及び変更、削除コマンドをCLIで操作した備忘録です!
* 構成する環境は下記の構成図の通りです！
* CLIインストール方法に関しては、お使いのPCによって異なりますので、[公式サイト](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html)を参考にインストールお願いします！

## 作成する構成図

![](images/kouseizu/all-kouseizu.png)

#### 前提事項

* CLIを操作しているIAMユーザは、”AdministratorFullAccess"権限のあるIAMユーザで操作しています。CLI操作権限があるIAMユーザを使用してください。
* 各リソースでパラメータを事前に環境変数を指定して作成する方法になります。
* EC2️からSSH接続する際、キーペアを作成して接続しています。
* EC2からRDSへの接続の際、mysqlクライアントをインストールして接続しています。
* ALBのリスナールールはHTTP80番で設定し、nginxをインストールし、稼働させることでHealthチャックを行うものとします。 

## 各種操作はこちらから

1. [CLIをインストール](cLI-command/cli-install.md)

2. [VPC環境を作成](cLI-command/cli-command-network.md)

3. [SGを作成](cLI-command/cli-command-SG.md)

4. [S3・EC2の作成](cLI-command/cli-command-S3-EC2.md)

5. [ALBの作成](cLI-command/cli-command-ALB.md)

6. [RDSの作成](cLI-command/cli-command-RDS.md)

7. [リソースを削除する](cLI-command/cli-delete-command.md)
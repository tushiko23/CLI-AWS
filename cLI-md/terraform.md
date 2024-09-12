# terraform環境構築
1. terraformをインストール
2. CLIをインストール
3. CLIに環境変数で認証情報を登録し、terraformの操作をできるようにする
4. 
## LocalPCにterraformをインストール
```
* パッケージリストを最新にする
```
sudo apt-get update && sudo apt-get upgrade -y
```
* Terraformをダウンロードするために、wgetとunzipをインストールする
```
sudo apt-get install wget unzip -y
```
* wgetを使用してterraformをインストール
```
wget https://releases.hashicorp.com/terraform/<バージョン>/terraform_<バージョン>_linux_amd64.zip
```
* 最新版をインストール(2024年9月地点では"1.9.5")

```
wget https://releases.hashicorp.com/terraform/1.9.5/terraform_1.9.5_linux_amd64.zip
```
* ダウンロードしたファイルを解凍
```
unzip terraform_<バージョン>_linux_amd64.zip
unzip terraform_1.9.5_linux_amd64.zip
```
* Terraformバイナリをグローバルに使用できるように、/usr/local/binに移動する
* Terraformを使用できるようにpassを通す
```
sudo mv terraform /usr/local/bin/
```
* Terraformのインストールを確認
```
terraform -v
```


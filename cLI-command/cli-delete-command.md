```
# 作成したIAMロールがどのインスタンスプロファイルにアタッチされているか確認
aws iam list-instance-profiles-for-role --role-name  tushiko-cli-role
```　

```
# インスタンスプロファイルの削除
ロールがEC2インスタンスプロファイルに関連付けられている場合、次のコマンドで先にインスタンスプロファイルからロールを削除。

```
aws iam remove-role-from-instance-profile \
  --instance-profile-name <インスタンスプロファイル名> \
  --role-name <アタッチしたロール名>

aws iam remove-role-from-instance-profile \
> --instance-profile-name "tushiko-cli-role-instance-profile" \
> --role-name tushiko-cli-role
```

# インスタンスプロファイル自体も削除
```
aws iam delete-instance-profile \
  --instance-profile-name <インスタンスプロファイル名>

aws iam delete-instance-profile \
> --instance-profile-name tushiko-cli-role-instance-profile
```
# インスタンスの終了

```
aws ec2 terminate-instances --instance-ids <インスタンスID>
```

1. インスタンスの詳細を確認するコマンド
describe-instances コマンドを使用して、EC2インスタンスの詳細を確認します。これにより、指定したインスタンスに関する全情報が返されます。

``
aws ec2 describe-instances --instance-ids <instance-id>
```
* <instance-id>: 取得したいインスタンスのIDを指定します。
* このコマンドでインスタンスの状態やタグ、パブリックIPなどの情報が表示。

2. インスタンスIDのみを取得するコマンド

```
aws ec2 describe-instances --query 'Reservations[*].Instances[*].InstanceId' --output text
```
直前で作成されたインスタンスのidを取得するコマンドでインスタンスの確認をする

* --query: JSON形式の出力から特定の情報を抜き出すために使用。
* --output text: 出力をテキスト形式で表示し、インスタンスIDのみ取得を

3. 直近に作成したインスタンスのIDを取得する
直近に作成したインスタンスのIDを取得したい場合は、launch-time に基づいてソートし、最新のインスタンスIDを取得する。

```
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].[InstanceId, LaunchTime]' \
  --output text | sort -k2 | tail -n 1 | awk '{print $1}'
```


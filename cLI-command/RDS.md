RDSの作成

1. SecretManagerに認証情報を保存する
2. サブネットグループの作成
3. RDSインスタンス作成

今回は、マスタユーザ名、マスタパスワードをSecretManagerに保存した認証情報を使用します
(費用を抑えたい方はマスタユーザ名・マスタパスワードを自己管理し入力してください)

1. 変数の設定
* シークレット名を設定
```
SECRET_NAME="mydbsecret"
```

* マスタユーザ名・マスタユーザパスワード名を設定
```
#設定したい"ユーザ名""マスタユーザ名"を設定
MASTER_USERNAME="admin"
MASTER_USER_PASSWORD="mypassword"
```

2. Secrets Managerに認証情報を保存する

```
aws secretsmanager create-secret \
    --name $SECRET_NAME \
    --description "RDS MySQL Master User credentials" \
    --secret-string '{"username":"'"$MASTER_USERNAME"'","password":"'"$MASTER_USER_PASSWORD"'"}'
```
* ARNの取得を確認
```
aws secretsmanager describe-secret --secret-id $SECRET_NAME --query 'ARN' --output text
```
* ARNはRDS作成時に使用するので変数に設定
```
SECRET_ARN=$(aws secretsmanager describe-secret --secret-id $SECRET_NAME --query 'ARN' --output text)
```
エビデンス画像

1. サブネットグループ作成
変数指定
```
SUBNET_GROUP_NAME="my-subnet-group"
```
* VPCタグ名
```
#自身が設定したvpcタグ
EC2_VPC_TAG_NAME='tushiko-cli-vpc'
```
* VPC ID
```
EC2_VPC_ID=$( \
  aws ec2 describe-vpcs \
    --filters Name=tag:Name,Values=${EC2_VPC_TAG_NAME}  \
    --query 'Vpcs[].VpcId' \
    --output text \
) \
  && echo ${EC2_VPC_ID}
#指定したVPC IDがでればOK
vpc-×××××××××××××××
```

* サブネットタグ名
ap-northeast-1aのプライベートサブネットのタグ名を指定。
ap-northeast-1cのプライベートサブネットのタグ名を指定。

```
EC2_SUBNET_TAG_NAME='tushiko-cli-private-subnet-a'
```
サブネットID

```
EC2_SUBNET_ID=$( \
  aws ec2 describe-subnets \
    --filters Name=vpc-id,Values=${EC2_VPC_ID} \
              Name=tag:Name,Values=${EC2_SUBNET_TAG_NAME} \
    --query "Subnets[].SubnetId" \
    --output text \
) \
&& echo ${EC2_SUBNET_ID}

#指定したサブネットIDが出ればOK
subnet-×××××××××××××
```

* サブネットIDの変数化 "ap-northeast-1a","ap-northeast-1c"のプライベートサブネットIDを指定
```
SUBNET_IDS="subnet-xxxxxxxx subnet-yyyyyyyy"
```

サブネットグループ作成

```
#--db-subnet-group-descriptionオプションでサブネットグループの説明
aws rds create-db-subnet-group \
    --db-subnet-group-name $SUBNET_GROUP_NAME \
    --db-subnet-group-description "My RDS Subnet Group"\
    --subnet-ids  $SUBNET_IDS          
```

コンソール上でも確認

RDS作成の変数を指定

"識別子名"・"データベースインスタンスクラス"・"データベースエンジン""ユーザ定義のDB名"
```
DB_INSTANCE_IDENTIFIER="mydbinstance"
DB_INSTANCE_CLASS="db.t3.micro"
DB_ENGINE="mysql"
DB_NAME="mydatabase"
MASTER_USERNAME="admin"
MASTER_USER_PASSWORD="mypassword"
```

* セキュリティグループ
```
#自身で作成したセキュリティグループ名
EC2_SECURITY_GROUP_NAME='tushiko-cli-sg2'
```
セキュリティグループID
```
EC2_SECURITY_GROUP_ID=$( \
  aws ec2 describe-security-groups \
    --filters Name=vpc-id,Values=${EC2_VPC_ID} \
              Name=group-name,Values=${EC2_SECURITY_GROUP_NAME} \
    --query 'SecurityGroups[].GroupId' \
    --output text \
) \
&& echo ${EC2_SECURITY_GROUP_ID}

#指定されたsg-idが出るか確認
sg-XXXXXXXXXXXXXXX
```

RDSの作成のコマンド

```
aws rds create-db-instance \
    --db-instance-identifier $DB_INSTANCE_IDENTIFIER \
    --db-instance-class $DB_INSTANCE_CLASS \
    --engine $DB_ENGINE \
    --db-name $DB_NAME \
    --db-subnet-group-name $SUBNET_GROUP_NAME \
    --vpc-security-group-ids $EC2_SECURITY_GROUP_ID \
    --allocated-storage 20 \
    --enable-iam-database-authentication \
    --master-username $MASTER_USERNAME \
    --master-user-password $MASTER_USER_PASSWORD \
    --availability-zone ap-northeast-1a
```

オプションの説明

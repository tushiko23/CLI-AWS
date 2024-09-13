AWS CLIで第5回環境の構築

# ALB構築
①ターゲットグループの作成
②ターゲットグループへターゲットの登録
③ELBの作成
④リスナー設定

①ターゲットグループの作成
変数を設定

```
## ターゲットグループ名
#設定したいターゲットグループを設定
TARGET_GROUP_NAME=tushiko-target-cli

## ターゲットグループのプロトコル
TARGET_GROUP_PROTCOL=HTTP

## ターゲットグループのポート
TARGET_GROUP_PORT=80

## ヘルスチェックパス
HTALTH_CHECK_PATH=/

## ヘルスチェックポート
HTALTH_CHECK_PORT=80

## ヘルスチェックプロトコル
HTALTH_CHECK_PROTCOL=HTTP

## ヘルスチェックの間隔(秒)
HTALTH_CHECK_INTERVAL=30

## ヘルスチェックのタイムアウト値(秒)
HTALTH_CHECK_TIMEOUT=5

## 正常と判断する閾値
HTALTHY_THRESHOLD_COUNT=5

## 異常と判断する閾値
UNHTALTHY_THRESHOLD_COUNT=2

## ターゲットのタイプ(lambdaも指定できるが今回は割愛)
## インスタンスを選択
TARGET_TYPE=instance

## VPCの関連付け
## VPCタグ名

#自身が設定したvpcタグ
EC2_VPC_TAG_NAME='tushiko-cli-vpc'

## VPC ID

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

②ターゲットグループの作成
```
aws elbv2 create-target-group \
--name ${TARGET_GROUP_NAME} \
--protocol ${TARGET_GROUP_PROTCOL} \
--port ${TARGET_GROUP_PORT} \
--health-check-path ${HTALTH_CHECK_PATH} \
--health-check-protocol  ${HTALTH_CHECK_PROTCOL} \
--health-check-port ${HTALTH_CHECK_PORT} \
--health-check-interval-seconds ${HTALTH_CHECK_INTERVAL} \
--health-check-timeout-seconds ${HTALTH_CHECK_TIMEOUT} \
--healthy-threshold-count ${HTALTHY_THRESHOLD_COUNT} \
--unhealthy-threshold-count ${UNHTALTHY_THRESHOLD_COUNT} \
--target-type ${TARGET_TYPE} \
--vpc-id ${EC2_VPC_ID}
```

* ターゲットグループの存在確認

```
aws elbv2 describe-target-groups --names ${TARGET_GROUP_NAME}
```

②ターゲット(今回はインスタンス)をターゲットグループに登録

インスタンスIDを取得　(今回は、直前で作成したインスタンスIDを確認するコマンドを使用して取得)

```
aws ec2 describe-instances \
  --query 'Reservations[*].Instances[*].[InstanceId, LaunchTime]' \
  --output text | sort -k2 | tail -n 1 | awk '{print $1}'
```
変数を指定
```
## ターゲットのインスタンスID(ターゲットタイプがinsntaceの場合のみ指定)
TARGET_ID=i-×××××××××××××××××
## ターゲットのポート
TARGET_PORT=80
## ELBの名前
## 自身が設定するELBを指定する
ELB_NAME=tushiko-elb-cli
```

```
* サブネットタグ名
ap-northeast-1aのパブリックサブネットのタグ名を指定。
```
EC2_SUBNET_TAG_NAME='tushiko-cli-public-subnet-a'
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


## ELBを作成するサブネットIDを指定(２つのAZのSubnetを指定)
```
##　取得したサブネットIDを指定
SUBETIDS="subnet-×××××××××××××××× subnet-××××××××××××××××"
```

関連付けるSGIDを取得
* ALBに関連付けるセキュリティグループ
```
##自身が作成したALBに関連付けるセキュリティグループID

EC2_SECURITY_GROUP_NAME='tushiko-cli-sg3'

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
```
## ELBにアタッチするSGID
##　出力したSGIDを登録
SGIDS=sg-XXXXXXXXXXXXXXX
## ELBのスキーマ
## internet-facingを選択
## SCHEME=internet-facing or internal
SCHEME=internet-facing

## ELBのタイプ
## applicationを選択
## ELB_TYPE=application or network
ELB_TYPE=application

## IPアドレスタイプ
IP_ADDRESS_TYPE=ipv4

## リスナーのプロトコル(HTTPの場合)
LISTENER_PROTCOL_HTTP=HTTP

## リスナーのプロトコル(HTTPの場合)
LISTENER_PORT_HTTP=80
```

②ターゲットの登録

* 作業前の状態確認

```
aws elbv2 describe-target-health \
--target-group-arn $(aws elbv2 describe-target-groups --query "TargetGroups[].TargetGroupArn" --output text --name ${TARGET_GROUP_NAME}) \
--query 'TargetHealthDescriptions[].Target' --output table
```

ターゲットの登録（ターゲットグループのタイプがinstanceの場合)

```
aws elbv2 register-targets \
--targets Id=${TARGET_ID},Port=${TARGET_PORT} \
--target-group-arn $(aws elbv2 describe-target-groups --query "TargetGroups[].TargetGroupArn" --output text --name ${TARGET_GROUP_NAME})
```
作業後の状態確認

```
aws elbv2 describe-target-health \
--target-group-arn $(aws elbv2 describe-target-groups --query "TargetGroups[].TargetGroupArn" --output text --name ${TARGET_GROUP_NAME}) \
--query 'TargetHealthDescriptions[].Target' --output table

## 以下が返ってくればOK
---------------------------------
|     DescribeTargetHealth      |
+----------------------+--------+
|          Id          | Port   |
+----------------------+--------+
|  i-××××××××××××××××× |  80    |
+----------------------+--------+
```

③ELB構築
* ELBが存在しない場合、エラーが返ってきます。

```
#作成状態確認
aws elbv2 describe-load-balancers --names ${ELB_NAME}
```

ELBの作成
```
aws elbv2 create-load-balancer \
--name ${ELB_NAME} \
--subnets ${SUBETIDS} \
--security-groups ${SGIDS} \
--scheme ${SCHEME} \
--type ${ELB_TYPE} \
--ip-address-type ${IP_ADDRESS_TYPE} \
--tags Key=Name,Value=${ELB_NAME}
```

ロードバランサーの作成を確認
```
aws elbv2 describe-load-balancers --names ${ELB_NAME}
```

④リスナー設定
作業前、リスナー登録(今回は、80番)があるか確認
```
aws elbv2 describe-listeners \
--load-balancer-arn $(aws elbv2 describe-load-balancers --query "LoadBalancers[].LoadBalancerArn" --output text --names ${ELB_NAME})
```

リスナーの設定(HTTPの場合)
```
aws elbv2 create-listener \
--load-balancer-arn $(aws elbv2 describe-load-balancers --query "LoadBalancers[].LoadBalancerArn" --output text --names ${ELB_NAME}) \
--protocol ${LISTENER_PROTCOL_HTTP} \
--port ${LISTENER_PORT_HTTP} \
--default-actions Type=forward,TargetGroupArn=$(aws elbv2 describe-target-groups --query "TargetGroups[].TargetGroupArn" \
--output text \
--name ${TARGET_GROUP_NAME})
```

* 作業後の状態確認
```
aws elbv2 describe-listeners \
--load-balancer-arn $(aws elbv2 describe-load-balancers --query "LoadBalancers[].LoadBalancerArn" --output text --names ${ELB_NAME})
```
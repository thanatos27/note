---
tags:
  - AWS
  - CloudFormation
  - IaC
created: 2026-05-27
updated: 2026-05-27
---

# AWS CloudFormation 入門リファレンス

## 目次

- [概要](#概要)
- [テンプレートの基本構造](#テンプレートの基本構造)
- [セクション詳細](#セクション詳細)
  - [Metadata（メタデータ）](#metadataメタデータ)
  - [Parameters（パラメータ）](#parametersパラメータ)
  - [Mappings（マッピング）](#mappingsマッピング)
  - [Rules（ルール）](#rulesルール)
  - [Conditions（条件）](#conditions条件)
  - [Resources（リソース）](#resourcesリソース)
  - [Outputs（出力）](#outputs出力)
  - [Transform（トランスフォーム）](#transformトランスフォーム)
- [組み込み関数](#組み込み関数)
- [疑似パラメータ](#疑似パラメータ)
- [よく使うリソースタイプ](#よく使うリソースタイプ)
  - [S3](#s3)
  - [Lambda](#lambda)
  - [API Gateway](#api-gateway)
  - [DynamoDB](#dynamodb)
  - [IAM](#iam)
  - [VPC・EC2](#vpcec2)
- [スタック操作（CLI）](#スタック操作cli)
- [変更セット（Change Sets）](#変更セットchange-sets)
- [スタック間の参照](#スタック間の参照)
- [ネストされたスタック](#ネストされたスタック)
- [ドリフト検出](#ドリフト検出)
- [StackSets](#stacksets)
- [削除ポリシー・更新ポリシー](#削除ポリシー更新ポリシー)
- [スタック保護・ロールバック復旧](#スタック保護ロールバック復旧)
- [検証・ガードレール](#検証ガードレール)
- [Tips・ベストプラクティス](#tipsベストプラクティス)
- [用語集](#用語集)
- [リンク集](#リンク集)

---

## 概要

AWS CloudFormation は JSON または YAML のテンプレートファイルで AWS リソースをコードとして定義・管理する IaC（Infrastructure as Code）サービス。

- **メリット**: AWS ネイティブ、追加ツール不要、多くの AWS リソースタイプに対応、ロールバック・ドリフト検出などの運用機能が充実
- **デメリット**: テンプレートが冗長になりやすい、プログラミング言語のような抽象化が難しい（CDK や SAM で補完可能）
- CDK は内部的に CloudFormation テンプレートを生成するため、CloudFormation の挙動を理解しておくと CDK も扱いやすくなる

---

## テンプレートの基本構造

```yaml
AWSTemplateFormatVersion: "2010-09-09"  # 現在サポートされている形式バージョン

Description: テンプレートの説明（省略可）

Metadata:          # テンプレートに関する追加情報（省略可）
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: "Network Configuration"
        Parameters:
          - Env

Parameters:        # デプロイ時に渡す変数（省略可）
  Env:
    Type: String
    AllowedValues: [dev, staging, prod]

Mappings:          # キーバリューの対応表（省略可）
  RegionMap:
    ap-northeast-1:
      AMI: ami-0b7546e839d7ace12

Conditions:        # リソース作成の条件式（省略可）
  IsProd: !Equals [!Ref Env, prod]

Resources:         # 作成するリソース（必須）
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-bucket

Rules:             # パラメータ間の入力検証（省略可）
  CheckEnv:
    Assertions:
      - Assert:
          Fn::Contains:
            - [dev, staging, prod]
            - !Ref Env
        AssertDescription: Env must be dev, staging, or prod

Transform: AWS::LanguageExtensions  # マクロ・拡張構文（省略可）

Outputs:           # 他スタックへのエクスポート、コンソール表示（省略可）
  BucketName:
    Value: !Ref MyBucket
    Export:
      Name: !Sub "${AWS::StackName}-BucketName"
```

---

## セクション詳細

### Metadata（メタデータ）

テンプレート全体の補足情報や、CloudFormation コンソールでのパラメータ表示制御に使う。

```yaml
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: "Network"
        Parameters:
          - VpcId
          - SubnetIds
    ParameterLabels:
      VpcId:
        default: "VPC ID"
```

`Metadata` は CloudFormation によって秘匿・変換されないため、パスワードやトークンを含めない。

### Parameters（パラメータ）

デプロイ時に動的に渡せる変数。

```yaml
Parameters:
  Env:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - staging
      - prod
    Description: デプロイ環境

  InstanceType:
    Type: String
    Default: t3.micro
    AllowedPattern: "t[23]\\.(micro|small|medium)"
    ConstraintDescription: "t2.* または t3.* のみ指定可能"

  DBPassword:
    Type: String
    NoEcho: true   # describe 系 API などではマスクされる

  SubnetIds:
    Type: List<AWS::EC2::Subnet::Id>   # AWS 固有型（プルダウンで選択可）
```

`NoEcho` は万能な秘匿機能ではない。`Metadata`、`Outputs`、リソースの `Metadata` ではマスクされず、リソースの主識別子に含めると平文が派生出力に現れる場合がある。シークレットは Secrets Manager や SSM Parameter Store の動的参照を使う。

**パラメータ型一覧（抜粋）**

| 型 | 説明 |
|----|------|
| `String` | 任意の文字列 |
| `Number` | 整数または浮動小数点数 |
| `List<Number>` | カンマ区切りの数値リスト |
| `CommaDelimitedList` | カンマ区切りの文字列リスト |
| `AWS::EC2::KeyPair::KeyName` | 既存のキーペア名 |
| `AWS::EC2::VPC::Id` | 既存の VPC ID |
| `AWS::EC2::Subnet::Id` | 既存のサブネット ID |
| `AWS::SSM::Parameter::Value<String>` | SSM Parameter Store の値を参照 |

### Mappings（マッピング）

リージョンや環境ごとに異なる値を定義した対応表。

```yaml
Mappings:
  EnvConfig:
    dev:
      InstanceType: t3.micro
      MinSize: 1
    prod:
      InstanceType: t3.medium
      MinSize: 2

  RegionToAMI:
    ap-northeast-1:
      AMI: ami-0b7546e839d7ace12
    us-east-1:
      AMI: ami-0c02fb55956c7d316

# 参照: !FindInMap [マップ名, キー1, キー2]
# 例: !FindInMap [EnvConfig, !Ref Env, InstanceType]
```

### Rules（ルール）

パラメータの組み合わせを、リソース作成前に検証する。単純な `AllowedValues` / `AllowedPattern` では表現しづらい入力制約に使う。

```yaml
Rules:
  RequireCertificateWhenHttps:
    RuleCondition: !Equals [!Ref EnableHttps, true]
    Assertions:
      - Assert: !Not [!Equals [!Ref CertificateArn, ""]]
        AssertDescription: HTTPS を有効にする場合は CertificateArn が必要
```

### Conditions（条件）

パラメータやマッピングの値に基づいてリソース作成を制御する。

```yaml
Conditions:
  IsProd: !Equals [!Ref Env, prod]
  IsNotProd: !Not [!Equals [!Ref Env, prod]]
  IsDevOrStaging: !Or
    - !Equals [!Ref Env, dev]
    - !Equals [!Ref Env, staging]

Resources:
  # 本番環境のみ作成
  ProdOnlyBucket:
    Type: AWS::S3::Bucket
    Condition: IsProd
    Properties:
      BucketName: prod-only-bucket

  # Condition で値を切り替える
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !If [IsProd, t3.medium, t3.micro]
```

### Resources（リソース）

テンプレート唯一の必須セクション。各リソースには一意の論理 ID を付ける。

```yaml
Resources:
  MyBucket:                 # 論理 ID（スタック内でユニーク）
    Type: AWS::S3::Bucket   # リソースタイプ
    DependsOn: AnotherResource  # 依存関係の明示（省略可）
    DeletionPolicy: Retain  # 削除ポリシー（省略可）
    UpdateReplacePolicy: Snapshot  # 置換時のポリシー（省略可）
    Properties:             # リソース固有の設定
      BucketName: my-bucket
      VersioningConfiguration:
        Status: Enabled
```

### Outputs（出力）

スタックの出力値。コンソールに表示したり、他スタックへのエクスポートに使う。

```yaml
Outputs:
  BucketName:
    Description: S3 バケット名
    Value: !Ref MyBucket

  BucketArn:
    Value: !GetAtt MyBucket.Arn
    Export:
      Name: !Sub "${AWS::StackName}-BucketArn"
```

`Outputs` にシークレット値を出力しない。`NoEcho` パラメータを参照しても `Outputs` ではマスクされない。

### Transform（トランスフォーム）

マクロや AWS 提供の拡張構文を適用する。SAM や `AWS::LanguageExtensions`、`AWS::Include` などで使う。

```yaml
Transform:
  - AWS::LanguageExtensions
  - AWS::Serverless-2016-10-31
```

`AWS::LanguageExtensions` を使うと `Fn::ForEach`、`Fn::Length`、`Fn::ToJsonString` などを利用できる。複数指定する場合、`AWS::LanguageExtensions` は `AWS::Serverless` より前に置く。

---

## 組み込み関数

| 関数 | 短縮形 | 説明 |
|------|--------|------|
| `Fn::Ref` | `!Ref` | リソースの ID またはパラメータ値を参照 |
| `Fn::GetAtt` | `!GetAtt` | リソースの属性値を取得（ARN など） |
| `Fn::Sub` | `!Sub` | 文字列に変数を埋め込む |
| `Fn::Join` | `!Join` | 区切り文字で文字列を結合 |
| `Fn::Select` | `!Select` | リストからインデックスで要素を取得 |
| `Fn::Split` | `!Split` | 文字列をリストに分割 |
| `Fn::FindInMap` | `!FindInMap` | Mappings から値を取得 |
| `Fn::If` | `!If` | 条件に応じて値を切り替える |
| `Fn::Equals` | `!Equals` | 2 つの値が等しいか比較 |
| `Fn::Not` | `!Not` | 条件を反転 |
| `Fn::And` | `!And` | 複数条件の AND |
| `Fn::Or` | `!Or` | 複数条件の OR |
| `Fn::ImportValue` | `!ImportValue` | 他スタックの Export を参照 |
| `Fn::Base64` | `!Base64` | Base64 エンコード（UserData など） |
| `Fn::Cidr` | `!Cidr` | CIDR ブロックを生成 |
| `Fn::Length` | - | リストの要素数（`AWS::LanguageExtensions` が必要） |
| `Fn::ToJsonString` | - | オブジェクトを JSON 文字列化（`AWS::LanguageExtensions` が必要） |

`Fn::Length`、`Fn::ToJsonString`、`Fn::ForEach` は `Transform: AWS::LanguageExtensions` を宣言したテンプレートで使う。これらの拡張関数は短縮形 YAML 構文を使えないため、`Fn::Length` のように明示的に書く。

```yaml
# !Ref: パラメータ → 値, リソース → 物理 ID
BucketName: !Ref MyBucket

# !GetAtt: リソースの属性
BucketArn: !GetAtt MyBucket.Arn
FunctionArn: !GetAtt MyFunction.Arn

# !Sub: 変数展開（${変数名}）
Name: !Sub "${AWS::StackName}-bucket"
PolicyArn: !Sub "arn:aws:iam::${AWS::AccountId}:policy/MyPolicy"

# !Join: 文字列結合
Tags: !Join [",", [tag1, tag2, tag3]]

# !Select: リストから要素取得
FirstAz: !Select [0, !GetAZs ""]

# !FindInMap: マッピング参照
AMI: !FindInMap [RegionToAMI, !Ref "AWS::Region", AMI]

# !If: 条件分岐
InstanceType: !If [IsProd, t3.medium, t3.micro]

# !ImportValue: 他スタックの出力参照
VpcId: !ImportValue "NetworkStack-VpcId"

# !Base64: EC2 UserData
UserData:
  Fn::Base64: |
    #!/bin/bash
    yum update -y

# Fn::Length: AWS::LanguageExtensions が必要
DelaySeconds:
  Fn::Length: !Ref QueueList
```

---

## 疑似パラメータ

テンプレート内で利用できる定義済み変数。

| 疑似パラメータ | 説明 | 例 |
|---------------|------|----|
| `AWS::AccountId` | AWSアカウントID | `123456789012` |
| `AWS::Region` | デプロイリージョン | `ap-northeast-1` |
| `AWS::StackName` | スタック名 | `my-stack` |
| `AWS::StackId` | スタックの ARN | `arn:aws:cloudformation:...` |
| `AWS::NoValue` | プロパティを省略（条件分岐で使用） | - |
| `AWS::Partition` | パーティション名 | `aws` / `aws-cn` / `aws-us-gov` |
| `AWS::URLSuffix` | ドメインサフィックス | `amazonaws.com` |

```yaml
# よく使う組み合わせ
BucketArn: !Sub "arn:${AWS::Partition}:s3:::${BucketName}"
LogGroupName: !Sub "/aws/lambda/${AWS::StackName}-function"
```

---

## よく使うリソースタイプ

### S3

```yaml
MyBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: !Sub "${AWS::StackName}-data"  # 省略で自動生成
    VersioningConfiguration:
      Status: Enabled
    BucketEncryption:
      ServerSideEncryptionConfiguration:
        - ServerSideEncryptionByDefault:
            SSEAlgorithm: AES256
    PublicAccessBlockConfiguration:
      BlockPublicAcls: true
      BlockPublicPolicy: true
      IgnorePublicAcls: true
      RestrictPublicBuckets: true
    LifecycleConfiguration:
      Rules:
        - Status: Enabled
          Transitions:
            - TransitionInDays: 30
              StorageClass: STANDARD_IA
          ExpirationInDays: 90

# バケットポリシー
MyBucketPolicy:
  Type: AWS::S3::BucketPolicy
  Properties:
    Bucket: !Ref MyBucket
    PolicyDocument:
      Statement:
        - Effect: Deny
          Principal: "*"
          Action: "s3:*"
          Resource:
            - !GetAtt MyBucket.Arn
            - !Sub "${MyBucket.Arn}/*"
          Condition:
            Bool:
              "aws:SecureTransport": false
```

### Lambda

```yaml
MyFunction:
  Type: AWS::Lambda::Function
  Properties:
    FunctionName: !Sub "${AWS::StackName}-function"
    Runtime: nodejs20.x
    Handler: index.handler
    Role: !GetAtt LambdaRole.Arn
    Timeout: 30
    MemorySize: 256
    Environment:
      Variables:
        TABLE_NAME: !Ref MyTable
        STAGE: !Ref Env
    Code:
      ZipFile: |
        exports.handler = async (event) => {
          return { statusCode: 200, body: 'Hello' };
        };
    # S3 からコードをデプロイする場合
    # Code:
    #   S3Bucket: my-deploy-bucket
    #   S3Key: functions/my-function.zip

LambdaRole:
  Type: AWS::IAM::Role
  Properties:
    AssumeRolePolicyDocument:
      Version: "2012-10-17"
      Statement:
        - Effect: Allow
          Principal:
            Service: lambda.amazonaws.com
          Action: sts:AssumeRole
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

### API Gateway

```yaml
MyApi:
  Type: AWS::ApiGateway::RestApi
  Properties:
    Name: !Sub "${AWS::StackName}-api"

ApiResource:
  Type: AWS::ApiGateway::Resource
  Properties:
    RestApiId: !Ref MyApi
    ParentId: !GetAtt MyApi.RootResourceId
    PathPart: items

ApiMethod:
  Type: AWS::ApiGateway::Method
  Properties:
    RestApiId: !Ref MyApi
    ResourceId: !Ref ApiResource
    HttpMethod: GET
    AuthorizationType: NONE
    Integration:
      Type: AWS_PROXY
      IntegrationHttpMethod: POST
      Uri: !Sub
        - "arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${Arn}/invocations"
        - Arn: !GetAtt MyFunction.Arn

ApiDeployment:
  Type: AWS::ApiGateway::Deployment
  DependsOn: ApiMethod
  Properties:
    RestApiId: !Ref MyApi
    StageName: !Ref Env

# Lambda への呼び出し許可
LambdaPermission:
  Type: AWS::Lambda::Permission
  Properties:
    FunctionName: !Ref MyFunction
    Action: lambda:InvokeFunction
    Principal: apigateway.amazonaws.com
    SourceArn: !Sub "arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${MyApi}/*/*"
```

### DynamoDB

```yaml
MyTable:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: !Sub "${AWS::StackName}-table"
    BillingMode: PAY_PER_REQUEST
    AttributeDefinitions:
      - AttributeName: pk
        AttributeType: S
      - AttributeName: sk
        AttributeType: S
      - AttributeName: gsi1pk
        AttributeType: S
    KeySchema:
      - AttributeName: pk
        KeyType: HASH
      - AttributeName: sk
        KeyType: RANGE
    GlobalSecondaryIndexes:
      - IndexName: GSI1
        KeySchema:
          - AttributeName: gsi1pk
            KeyType: HASH
        Projection:
          ProjectionType: ALL
    PointInTimeRecoverySpecification:
      PointInTimeRecoveryEnabled: true
    SSESpecification:
      SSEEnabled: true
```

### IAM

```yaml
MyRole:
  Type: AWS::IAM::Role
  Properties:
    RoleName: !Sub "${AWS::StackName}-role"
    AssumeRolePolicyDocument:
      Version: "2012-10-17"
      Statement:
        - Effect: Allow
          Principal:
            Service: lambda.amazonaws.com
          Action: sts:AssumeRole
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
    Policies:
      - PolicyName: DynamoDBAccess
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
            - Effect: Allow
              Action:
                - dynamodb:GetItem
                - dynamodb:PutItem
                - dynamodb:Query
              Resource:
                - !GetAtt MyTable.Arn
                - !Sub "${MyTable.Arn}/index/*"
```

### VPC・EC2

```yaml
MyVpc:
  Type: AWS::EC2::VPC
  Properties:
    CidrBlock: 10.0.0.0/16
    EnableDnsHostnames: true
    EnableDnsSupport: true

PublicSubnet:
  Type: AWS::EC2::Subnet
  Properties:
    VpcId: !Ref MyVpc
    CidrBlock: 10.0.0.0/24
    AvailabilityZone: !Select [0, !GetAZs ""]
    MapPublicIpOnLaunch: true

InternetGateway:
  Type: AWS::EC2::InternetGateway

VpcGatewayAttachment:
  Type: AWS::EC2::VPCGatewayAttachment
  Properties:
    VpcId: !Ref MyVpc
    InternetGatewayId: !Ref InternetGateway

SecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: Web server SG
    VpcId: !Ref MyVpc
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 443
        ToPort: 443
        CidrIp: 0.0.0.0/0
```

---

## スタック操作（CLI）

```bash
# テンプレートを検証
aws cloudformation validate-template --template-body file://template.yaml

# スタック作成
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=Env,ParameterValue=dev \
  --capabilities CAPABILITY_IAM  # IAM リソースを含む場合に必要

# スタック更新
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=Env,UsePreviousValue=true \
  --capabilities CAPABILITY_IAM

# スタック削除
aws cloudformation delete-stack --stack-name my-stack

# スタックのイベントを表示（トラブルシュート時）
aws cloudformation describe-stack-events --stack-name my-stack

# スタックのリソース一覧
aws cloudformation list-stack-resources --stack-name my-stack

# スタックの出力を確認
aws cloudformation describe-stacks --stack-name my-stack \
  --query "Stacks[0].Outputs"

# スタック一覧
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE
```

**主要な capabilities**

| capabilities | 必要なケース |
|-------------|-------------|
| `CAPABILITY_IAM` | IAM リソース（ロール・ポリシー）を作成する |
| `CAPABILITY_NAMED_IAM` | `RoleName` / `PolicyName` など、名前付き IAM リソースを作成する |
| `CAPABILITY_AUTO_EXPAND` | マクロ（SAM Transform など）を含む |

名前を明示した IAM リソースを含む場合は、`CAPABILITY_IAM` ではなく `CAPABILITY_NAMED_IAM` を指定する。迷う場合は変更セットで必要な capability を確認する。

---

## 変更セット（Change Sets）

本番への変更前に差分を確認するための仕組み。実際のリソース変更は行われない。

```bash
# 変更セットを作成
aws cloudformation create-change-set \
  --stack-name my-stack \
  --change-set-name my-changeset \
  --template-body file://template.yaml \
  --parameters ParameterKey=Env,ParameterValue=prod \
  --capabilities CAPABILITY_IAM

# 変更セットの内容を確認
aws cloudformation describe-change-set \
  --stack-name my-stack \
  --change-set-name my-changeset

# 変更セットを実行（実際にデプロイ）
aws cloudformation execute-change-set \
  --stack-name my-stack \
  --change-set-name my-changeset

# 変更セットを削除（実行しない場合）
aws cloudformation delete-change-set \
  --stack-name my-stack \
  --change-set-name my-changeset
```

変更セットでは各リソースの `Action`（Add / Modify / Remove）と `Replacement`（True / False / Conditional）を確認する。`Replacement: True` はリソースの再作成を意味するため、データベースやストレージの変更には特に注意する。

---

## スタック間の参照

**エクスポート側**

```yaml
Outputs:
  VpcId:
    Value: !Ref MyVpc
    Export:
      Name: !Sub "${AWS::StackName}-VpcId"

  SubnetId:
    Value: !Ref PublicSubnet
    Export:
      Name: !Sub "${AWS::StackName}-SubnetId"
```

**インポート側**

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      SubnetId: !ImportValue "NetworkStack-SubnetId"
```

エクスポートされた値を参照しているスタックが存在する間は、エクスポート元のスタックを削除・変更できない。疎結合を保ちたい場合は SSM Parameter Store 経由での参照も有効。

### 動的参照

Secrets Manager や SSM Parameter Store の値をテンプレートに直接埋め込まず、デプロイ時に解決する。

```yaml
Parameters:
  DbPasswordParameter:
    Type: AWS::SSM::Parameter::Name

Resources:
  MyDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      MasterUsername: admin
      MasterUserPassword: !Sub "{{resolve:ssm-secure:${DbPasswordParameter}:1}}"
```

動的参照はテンプレート内の秘密情報を減らすための仕組みだが、リソースの主識別子や `Outputs` に秘密値を出さない設計は別途必要。

---

## ネストされたスタック

大きなテンプレートをモジュールに分割できる。子スタックのテンプレートは S3 に配置する必要がある。

```yaml
# 親スタック
NetworkStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: https://s3.amazonaws.com/my-bucket/network.yaml
    Parameters:
      Env: !Ref Env
    TimeoutInMinutes: 30

AppStack:
  Type: AWS::CloudFormation::Stack
  DependsOn: NetworkStack
  Properties:
    TemplateURL: https://s3.amazonaws.com/my-bucket/app.yaml
    Parameters:
      VpcId: !GetAtt NetworkStack.Outputs.VpcId
```

ネストされたスタックの `Outputs` は `!GetAtt StackName.Outputs.OutputKey` で参照できる。

---

## ドリフト検出

CloudFormation 外でリソースが変更された場合（コンソール直接操作など）に差分を検出する機能。

```bash
# ドリフト検出を開始
aws cloudformation detect-stack-drift --stack-name my-stack

# ドリフト検出の結果を確認
aws cloudformation describe-stack-drift-detection-status \
  --stack-drift-detection-id <detection-id>

# リソースごとのドリフト詳細を確認
aws cloudformation describe-stack-resource-drifts \
  --stack-name my-stack \
  --stack-resource-drift-status-filters MODIFIED DELETED
```

ドリフト検出後の対処:
1. テンプレートを実際のリソース状態に合わせて修正し、スタックを更新する
2. または `update-stack` でテンプレートの定義状態に戻す

---

## StackSets

複数の AWS アカウント・リージョンに同じスタックを一括デプロイする機能。

```bash
# StackSet を作成
aws cloudformation create-stack-set \
  --stack-set-name my-stackset \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_IAM

# スタックインスタンスを追加（アカウント + リージョン指定）
aws cloudformation create-stack-instances \
  --stack-set-name my-stackset \
  --accounts 111111111111 222222222222 \
  --regions ap-northeast-1 us-east-1 \
  --parameter-overrides ParameterKey=Env,ParameterValue=prod
```

AWS Organizations 連携を使うと、OU（組織単位）単位で自動デプロイができる。

---

## 削除ポリシー・更新ポリシー

### DeletionPolicy

スタック削除時のリソースの挙動を制御する。

| 値 | 挙動 |
|----|------|
| `Delete`（デフォルト） | スタック削除時にリソースも削除 |
| `Retain` | スタック削除時もリソースを保持 |
| `RetainExceptOnCreate` | 通常は保持するが、作成操作のロールバック時だけ削除 |
| `Snapshot` | 削除前にスナップショットを作成（RDS・EBS・ElastiCache など） |

`Delete` が基本のデフォルトだが、`AWS::RDS::DBCluster` と、`DBClusterIdentifier` を指定しない一部の `AWS::RDS::DBInstance` はデフォルトが `Snapshot`。また `Snapshot` は対応リソースでのみ使える。

```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  DeletionPolicy: Snapshot
  Properties:
    ...
```

### UpdateReplacePolicy

更新によってリソースが置換（再作成）される場合の挙動を制御する。

```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  UpdateReplacePolicy: Snapshot  # 置換前にスナップショット
  DeletionPolicy: Retain
  Properties:
    ...
```

### DependsOn

リソースの作成・更新・削除の順序を明示的に制御する。通常は CloudFormation が `!Ref` / `!GetAtt` から依存関係を自動推定するため、参照関係のないリソース間にのみ記述する。

```yaml
AppServer:
  Type: AWS::EC2::Instance
  DependsOn:
    - DatabaseInstance
    - NatGateway
  Properties:
    ...
```

---

## スタック保護・ロールバック復旧

### Termination Protection

誤削除を防ぐため、重要なスタックには終了保護を有効化する。

```bash
aws cloudformation update-termination-protection \
  --stack-name prod-stack \
  --enable-termination-protection
```

### Stack Policy

スタック更新時に、重要リソースの更新・置換・削除を防ぐための JSON ポリシー。RDS などの状態を持つリソースに有効。

```json
{
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "Update:*",
      "Principal": "*",
      "Resource": "LogicalResourceId/ProductionDatabase"
    }
  ]
}
```

### ロールバック復旧

更新失敗後に `UPDATE_ROLLBACK_FAILED` になると、通常の更新ができない。原因を修正したうえで `continue-update-rollback` を実行する。

```bash
aws cloudformation continue-update-rollback --stack-name my-stack
```

---

## 検証・ガードレール

| ツール | 用途 |
|--------|------|
| `validate-template` | テンプレート構文の基本検証 |
| `cfn-lint` | リソース仕様、プロパティ、依存関係の静的チェック |
| CloudFormation Guard | 組織ルールやセキュリティ基準への適合チェック |
| CloudFormation Hooks | スタック作成・更新・削除前にポリシー検査を実行 |

```bash
# cfn-lint
cfn-lint template.yaml

# CloudFormation Guard
cfn-guard validate --rules rules.guard --data template.yaml
```

本番用テンプレートは、少なくとも `validate-template` と `cfn-lint` を CI で実行する。

---

## Tips・ベストプラクティス

| カテゴリ | 内容 |
|----------|------|
| **フォーマット** | YAML を推奨（JSON より可読性が高く、コメントが書ける） |
| **テンプレート分割** | 1 テンプレート 1 スタックを基本に、大きくなったらネストや別スタックに分割 |
| **命名** | `!Sub "${AWS::StackName}-resource"` でリソース名にスタック名を含めると環境が混在しない |
| **DeletionPolicy** | データを持つリソースには `Retain` を検討し、RDS・EBS など Snapshot 対応リソースでは `Snapshot` も検討する |
| **変更セット** | 本番への変更は変更セットで事前確認。`Replacement: True` のリソースに注意 |
| **capabilities** | IAM リソースには必ず適切な `--capabilities` フラグを付ける |
| **ドリフト** | 定期的にドリフト検出を実行し、コンソール直接操作を発見・修正する |
| **シークレット** | テンプレートやパラメータにシークレット値を含めない。Secrets Manager / SSM Parameter Store の動的参照を使う |
| **パラメータ制限** | `AllowedValues` / `AllowedPattern` で不正入力を防ぐ |
| **大きなテンプレート** | `--template-body` は 51,200 bytes、S3 URL 経由は 1 MB、リソース数は 1 スタックあたり 500 |

---

## 用語集

| 用語 | 説明 |
|------|------|
| **スタック** | CloudFormation が管理するリソースの単位。テンプレートから作成される |
| **テンプレート** | リソースを定義する JSON / YAML ファイル |
| **論理 ID** | テンプレート内でリソースを識別する名前（スタック内でユニーク） |
| **物理 ID** | 実際に AWS に作成されたリソースの ID（バケット名、インスタンス ID など） |
| **変更セット** | スタック更新の差分をデプロイ前に確認するための仕組み |
| **ドリフト** | CloudFormation 外でリソースが変更された状態 |
| **組み込み関数** | テンプレート内で使える `!Ref` や `!GetAtt` などの関数 |
| **疑似パラメータ** | `AWS::Region` など、定義済みの環境変数 |
| **Export / Import** | スタック間で値を受け渡す仕組み（`Outputs.Export` と `!ImportValue`） |
| **ネストされたスタック** | 親スタックから子スタックを呼び出す構成 |
| **StackSet** | 複数アカウント・リージョンへの一括デプロイ |
| **DeletionPolicy** | スタック削除時のリソース挙動を制御するプロパティ |
| **capabilities** | IAM やマクロを含むスタックのデプロイ時に必要な許可フラグ |
| **Transform** | SAM（Serverless Application Model）などのマクロを展開する宣言 |

---

## リンク集

- [CloudFormation 公式ドキュメント](https://docs.aws.amazon.com/cloudformation/index.html)
- [テンプレートリファレンス（リソースタイプ一覧）](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
- [組み込み関数リファレンス](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)
- [CloudFormation の制限](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html)
- [AWS SAM（Serverless Application Model）](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/)

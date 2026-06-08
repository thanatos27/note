---
tags:
  - AWS
  - CDK
  - IaC
  - TypeScript
created: 2026-05-26
updated: 2026-05-27
---

# AWS CDK 入門リファレンス

## 目次

- [概要](#概要)
- [セットアップ](#セットアップ)
- [基本概念](#基本概念)
  - [App・Stack・Construct](#appstackconstruct)
  - [Construct レベル（L1/L2/L3）](#construct-レベルl1l2l3)
- [プロジェクト構成](#プロジェクト構成)
- [よく使うコンストラクト](#よく使うコンストラクト)
  - [S3](#s3)
  - [Lambda](#lambda)
  - [API Gateway](#api-gateway)
  - [DynamoDB](#dynamodb)
  - [VPC・EC2](#vpcec2)
  - [IAM](#iam)
- [環境・コンテキスト](#環境コンテキスト)
- [Assets・Bundling](#assetsbundling)
- [CDK CLI コマンド](#cdk-cli-コマンド)
- [スタック間の参照](#スタック間の参照)
- [既存リソースの参照・取り込み](#既存リソースの参照取り込み)
- [Stage・複数環境](#stage複数環境)
- [カスタムコンストラクト](#カスタムコンストラクト)
- [Aspects（横断的な変更）](#aspects横断的な変更)
- [テスト](#テスト)
- [CI/CD](#cicd)
- [セキュリティ](#セキュリティ)
- [論理 ID とリファクタリング](#論理-id-とリファクタリング)
- [Tips・ベストプラクティス](#tipsベストプラクティス)
- [用語集](#用語集)
- [リンク集](#リンク集)

---

## 概要

AWS CDK（Cloud Development Kit）は TypeScript / Python / Java などのプログラミング言語で AWS インフラを定義する IaC ツール。内部的には CloudFormation テンプレートを生成する。

- **メリット**: ロジック・ループ・抽象化をコードで書ける、型補完が効く、テスト可能
- **デメリット**: CDK バージョンアップで破壊的変更が来ることがある、CloudFormation の制約を受ける
- このメモは **CDK v2（aws-cdk-lib）** を前提としている

---

## セットアップ

```bash
# CDK CLI のインストール
npm install -g aws-cdk

# バージョン確認
cdk --version

# 新規プロジェクト作成（TypeScript）
mkdir my-cdk-app && cd my-cdk-app
cdk init app --language typescript

# AWS 認証情報の設定（事前に必要）
aws configure
# または環境変数: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_DEFAULT_REGION

# 対象の account + region ごとに、CDK 用 S3・ECR・IAM Role などをプロビジョニング
cdk bootstrap aws://<ACCOUNT_ID>/<REGION>
# 例: cdk bootstrap aws://123456789012/ap-northeast-1
```

`cdk bootstrap` は通常、デプロイ先の AWS アカウント・リージョンごとに一度実行する。CDK のバージョンや利用機能によって bootstrap stack の更新が必要になることもある。

---

## 基本概念

### App・Stack・Construct

```
App
└── Stack（CloudFormation スタック単位）
    └── Construct（リソースの構成単位）
        └── Construct（ネスト可能）
```

```typescript
import * as cdk from 'aws-cdk-lib';
import { MyStack } from './my-stack';

const app = new cdk.App();
new MyStack(app, 'MyStack', {
  env: { account: '123456789012', region: 'ap-northeast-1' },
});
```

```typescript
// lib/my-stack.ts
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';

export class MyStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyBucket', {
      versioned: true,
    });
  }
}
```

### Construct レベル（L1/L2/L3）

| レベル | 名称 | 特徴 | 例 |
|--------|------|------|----|
| L1 | Cfn（CloudFormation）リソース | CloudFormation リソースと 1:1 対応、プロパティ名が CF と同じ | `CfnBucket` |
| L2 | デフォルトの高レベル Construct | デフォルト値・メソッドが充実、型安全 | `Bucket`, `Function` |
| L3 | Patterns | 複数リソースをまとめた定番構成 | `ApplicationLoadBalancedFargateService` |

```typescript
// L1: CloudFormation プロパティを直接指定
import { aws_s3 as s3 } from 'aws-cdk-lib';
new s3.CfnBucket(this, 'L1Bucket', {
  versioningConfiguration: { status: 'Enabled' },
});

// L2: 便利なメソッド・プロパティ付き（通常こちらを使う）
new s3.Bucket(this, 'L2Bucket', {
  versioned: true,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
});
```

---

## プロジェクト構成

```
my-cdk-app/
├── bin/
│   └── my-cdk-app.ts      # App エントリポイント
├── lib/
│   └── my-cdk-app-stack.ts # Stack 定義
├── test/
│   └── my-cdk-app.test.ts  # テスト
├── cdk.json                 # CDK 設定
├── package.json
└── tsconfig.json
```

`cdk.json` の主要設定:

```json
{
  "app": "npx ts-node --prefer-ts-exts bin/my-cdk-app.ts",
  "context": {
    "@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId": true
  }
}
```

---

## よく使うコンストラクト

### S3

```typescript
import * as s3 from 'aws-cdk-lib/aws-s3';

const bucket = new s3.Bucket(this, 'MyBucket', {
  bucketName: 'my-unique-bucket-name',   // 省略で自動生成
  versioned: true,
  encryption: s3.BucketEncryption.S3_MANAGED,
  blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
  removalPolicy: cdk.RemovalPolicy.DESTROY,     // スタック削除時にバケットも削除
  autoDeleteObjects: true,                       // オブジェクトも一緒に削除
  lifecycleRules: [
    {
      expiration: cdk.Duration.days(90),
      transitions: [
        {
          storageClass: s3.StorageClass.INFREQUENT_ACCESS,
          transitionAfter: cdk.Duration.days(30),
        },
      ],
    },
  ],
});
```

空でない S3 バケットは通常削除できないため、開発環境でスタック削除時にバケットごと消したい場合は `removalPolicy: DESTROY` と `autoDeleteObjects: true` を組み合わせる。本番では誤削除を避けるため `RETAIN` を検討する。

### Lambda

```typescript
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as nodejs from 'aws-cdk-lib/aws-lambda-nodejs';

// Node.js Lambda（esbuild でバンドル）
const fn = new nodejs.NodejsFunction(this, 'MyFunction', {
  entry: 'src/handlers/my-handler.ts',
  handler: 'handler',
  runtime: lambda.Runtime.NODEJS_20_X,
  timeout: cdk.Duration.seconds(30),
  memorySize: 256,
  environment: {
    TABLE_NAME: table.tableName,
  },
});

// S3 バケットへの読み取り権限を付与
bucket.grantRead(fn);
```

### API Gateway

```typescript
import * as apigw from 'aws-cdk-lib/aws-apigateway';

const api = new apigw.RestApi(this, 'MyApi', {
  restApiName: 'My Service',
  defaultCorsPreflightOptions: {
    allowOrigins: apigw.Cors.ALL_ORIGINS,
    allowMethods: apigw.Cors.ALL_METHODS,
  },
});

const items = api.root.addResource('items');
items.addMethod('GET', new apigw.LambdaIntegration(fn));
items.addMethod('POST', new apigw.LambdaIntegration(fn));
```

### DynamoDB

```typescript
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';

const table = new dynamodb.Table(this, 'MyTable', {
  tableName: 'MyTable',
  partitionKey: { name: 'pk', type: dynamodb.AttributeType.STRING },
  sortKey:      { name: 'sk', type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
  pointInTimeRecoverySpecification: {
    pointInTimeRecoveryEnabled: true,
  },
});

// GSI（Global Secondary Index）
table.addGlobalSecondaryIndex({
  indexName: 'GSI1',
  partitionKey: { name: 'gsi1pk', type: dynamodb.AttributeType.STRING },
  sortKey:      { name: 'gsi1sk', type: dynamodb.AttributeType.STRING },
});

// Lambda に読み書き権限
table.grantReadWriteData(fn);
```

### VPC・EC2

```typescript
import * as ec2 from 'aws-cdk-lib/aws-ec2';

// VPC（明示的に Public/Private/Isolated サブネットを作成）
const vpc = new ec2.Vpc(this, 'MyVpc', {
  maxAzs: 2,
  natGateways: 1,
  subnetConfiguration: [
    { name: 'Public',   subnetType: ec2.SubnetType.PUBLIC },
    { name: 'Private',  subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS },
    { name: 'Isolated', subnetType: ec2.SubnetType.PRIVATE_ISOLATED },
  ],
});

// subnetConfiguration を省略すると、デフォルトでは AZ ごとに Public と Private のサブネットが作成される

// セキュリティグループ
const sg = new ec2.SecurityGroup(this, 'MySg', { vpc });
sg.addIngressRule(ec2.Peer.anyIpv4(), ec2.Port.tcp(443));
```

### IAM

```typescript
import * as iam from 'aws-cdk-lib/aws-iam';

// ポリシーを直接付与
fn.addToRolePolicy(new iam.PolicyStatement({
  actions: ['s3:GetObject', 's3:PutObject'],
  resources: [bucket.arnForObjects('*')],
}));

// マネージドポリシーを付与
fn.role?.addManagedPolicy(
  iam.ManagedPolicy.fromAwsManagedPolicyName('AmazonSQSFullAccess')
);
```

---

## 環境・コンテキスト

```typescript
// env を明示指定
new MyStack(app, 'MyStack', {
  env: {
    account: process.env.CDK_DEFAULT_ACCOUNT,
    region: process.env.CDK_DEFAULT_REGION,
  },
});

// cdk.json の context から値を取得
const stage = app.node.tryGetContext('stage') ?? 'dev';

// コマンドラインから渡す
// cdk deploy -c stage=prod
```

`CDK_DEFAULT_ACCOUNT` / `CDK_DEFAULT_REGION` は現在の AWS 認証情報に応じて値が変わるため、個人開発や検証では便利。本番環境では意図しないアカウント・リージョンへのデプロイを避けるため、固定値や明示的な設定ファイルから渡す方が安全。

`cdk.context.json` は `Vpc.fromLookup()` などの lookup 結果をキャッシュするファイル。再現性のため通常はコミット対象にする。lookup 結果を更新したい場合は `cdk context` でキーを確認し、`cdk context --clear` や `cdk context --reset <KEY>` を使う。

---

## Assets・Bundling

CDK は Lambda のコード、Docker イメージ、静的ファイルなどを asset として扱い、bootstrap で作成された S3 バケットや ECR リポジトリへアップロードする。

```typescript
import * as lambda from 'aws-cdk-lib/aws-lambda';

new lambda.Function(this, 'PlainFunction', {
  runtime: lambda.Runtime.NODEJS_20_X,
  handler: 'index.handler',
  code: lambda.Code.fromAsset('lambda'),
});
```

`NodejsFunction` は esbuild を使って TypeScript / JavaScript を bundle する。依存パッケージを含めるか外部化するかは `bundling` で制御できる。

```typescript
import * as nodejs from 'aws-cdk-lib/aws-lambda-nodejs';

new nodejs.NodejsFunction(this, 'BundledFunction', {
  entry: 'src/handlers/my-handler.ts',
  runtime: lambda.Runtime.NODEJS_20_X,
  bundling: {
    minify: true,
    sourceMap: true,
    externalModules: ['@aws-sdk/*'],
  },
});
```

---

## CDK CLI コマンド

```bash
# テンプレート生成・確認（デプロイはしない）
cdk synth

# 差分確認
cdk diff

# デプロイ
cdk deploy

# 複数スタックをまとめてデプロイ
cdk deploy --all

# スタック削除
cdk destroy

# スタック一覧
cdk list

# CloudFormation テンプレートを JSON で出力
cdk synth --json > template.json

# ホットスワップデプロイ（一部リソースのみ、高速）
cdk deploy --hotswap

# ウォッチモード（変更を検知して自動デプロイ。デフォルトで hotswap を使う）
cdk watch
```

`--hotswap` は CloudFormation を介さず、対応リソースだけを直接更新する。Lambda、ECS、Step Functions、S3 bucket deployments、CodeBuild、AppSync、API Gateway、Bedrock、EventBridge、DynamoDB、SQS、CloudWatch などが対象。本番では CloudFormation stack と実リソースの差分が残り得るため使わない。

`cdk watch` はデフォルトで hotswap を使う。hotswap 非対応の変更も通常の CloudFormation デプロイへフォールバックしたい場合は `--hotswap-fallback` を付ける。

---

## スタック間の参照

```typescript
// スタック A で construct を公開
export class StackA extends cdk.Stack {
  public readonly bucket: s3.Bucket;
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
    this.bucket = new s3.Bucket(this, 'SharedBucket');
  }
}

// スタック B で props 経由で受け取る
interface StackBProps extends cdk.StackProps {
  bucket: s3.Bucket;
}

export class StackB extends cdk.Stack {
  constructor(scope: Construct, id: string, props: StackBProps) {
    super(scope, id, props);
    props.bucket.grantRead(someFunction);
  }
}

// bin/app.ts
const stackA = new StackA(app, 'StackA');
new StackB(app, 'StackB', { bucket: stackA.bucket });
```

同じ CDK app 内で construct や attribute を渡すと、必要に応じて CDK が CloudFormation の Export / ImportValue を生成する。明示的に CloudFormation export/import を扱いたい場合は `CfnOutput` と `Fn.importValue` を使う。

```typescript
new cdk.CfnOutput(this, 'BucketNameOutput', {
  value: bucket.bucketName,
  exportName: 'SharedBucketName',
});

const bucketName = cdk.Fn.importValue('SharedBucketName');
```

---

## 既存リソースの参照・取り込み

既に存在する AWS リソースを CDK から参照する場合は `fromXxxArn` / `fromXxxName` / `fromLookup` を使う。

```typescript
const bucket = s3.Bucket.fromBucketName(this, 'ImportedBucket', 'existing-bucket-name');
const vpc = ec2.Vpc.fromLookup(this, 'ImportedVpc', {
  vpcName: 'main-vpc',
});
```

`fromXxx` で参照したリソースは CDK 管理下に移るわけではない。リソース自体を CloudFormation / CDK 管理に取り込みたい場合は `cdk import` や `cdk deploy --import-existing-resources` を検討する。

---

## Stage・複数環境

複数環境では、環境ごとに stack 名・account・region・設定値を明示する。単純な構成なら stack を複数作るだけでよい。

```typescript
interface MyStackProps extends cdk.StackProps {
  stageName: string;
}

new MyStack(app, 'MyAppDev', {
  env: { account: '111111111111', region: 'ap-northeast-1' },
  stageName: 'dev',
});

new MyStack(app, 'MyAppProd', {
  env: { account: '222222222222', region: 'ap-northeast-1' },
  stageName: 'prod',
});
```

複数 stack をまとめた単位を作りたい場合は `cdk.Stage` を使う。

```typescript
class MyApplicationStage extends cdk.Stage {
  constructor(scope: Construct, id: string, props: cdk.StageProps) {
    super(scope, id, props);
    new ApiStack(this, 'Api');
    new DatabaseStack(this, 'Database');
  }
}
```

---

## カスタムコンストラクト

```typescript
import { Construct } from 'constructs';
import * as lambda from 'aws-cdk-lib/aws-lambda-nodejs';
import * as apigw from 'aws-cdk-lib/aws-apigateway';

interface ApiLambdaProps {
  entry: string;
  environment?: Record<string, string>;
}

// 再利用可能な L3 相当のカスタムコンストラクト
class ApiLambda extends Construct {
  public readonly fn: lambda.NodejsFunction;
  public readonly api: apigw.RestApi;

  constructor(scope: Construct, id: string, props: ApiLambdaProps) {
    super(scope, id);

    this.fn = new lambda.NodejsFunction(this, 'Function', {
      entry: props.entry,
      environment: props.environment,
    });

    this.api = new apigw.RestApi(this, 'Api');
    this.api.root.addMethod('ANY', new apigw.LambdaIntegration(this.fn));
  }
}
```

---

## Aspects（横断的な変更）

スタック内の全リソースに対して一括で変更を適用したい場合に使う。

```typescript
import { IAspect } from 'aws-cdk-lib';
import { IConstruct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';

// 全 S3 バケットにバージョニングを強制する Aspect
class EnforceS3Versioning implements IAspect {
  visit(node: IConstruct): void {
    if (node instanceof s3.CfnBucket) {
      node.versioningConfiguration = { status: 'Enabled' };
    }
  }
}

// スタックに適用
cdk.Aspects.of(stack).add(new EnforceS3Versioning());
```

---

## テスト

```typescript
// test/my-stack.test.ts
import { App } from 'aws-cdk-lib';
import { Template } from 'aws-cdk-lib/assertions';
import { MyStack } from '../lib/my-stack';

describe('MyStack', () => {
  let template: Template;

  beforeEach(() => {
    const app = new App();
    const stack = new MyStack(app, 'TestStack');
    template = Template.fromStack(stack);
  });

  test('S3 バケットが作成される', () => {
    template.hasResourceProperties('AWS::S3::Bucket', {
      VersioningConfiguration: { Status: 'Enabled' },
    });
  });

  test('Lambda 関数が 1 つだけ作成される', () => {
    template.resourceCountIs('AWS::Lambda::Function', 1);
  });

  test('スナップショットテスト', () => {
    expect(template.toJSON()).toMatchSnapshot();
  });
});
```

```bash
# テスト実行
npm test
```

代表的なテストの種類:

| 種類 | 内容 |
|------|------|
| **Snapshot test** | 生成された CloudFormation テンプレート全体の差分を見る |
| **Fine-grained assertion** | `hasResourceProperties` などで重要なプロパティだけ検証する |
| **Validation** | 必須 props や不正な組み合わせを construct 側で検証する |

---

## CI/CD

CI では `synth` とテストを必ず実行し、デプロイ前に `diff` を確認できる形にする。

```bash
npm ci
npm run build
npm test
cdk synth
cdk diff
cdk deploy --require-approval never
```

GitHub Actions などから AWS に接続する場合は、長期アクセスキーより OIDC で AssumeRole する構成が推奨。CDK 自体でパイプラインを定義したい場合は CDK Pipelines も選択肢になる。

---

## セキュリティ

| 観点 | 注意点 |
|------|--------|
| **IAM** | `grantRead` などの grant メソッドを優先し、必要な権限だけを付与する |
| **Managed policy** | `AmazonSQSFullAccess` のような広い権限は検証用途に留め、本番では絞った policy を作る |
| **Secrets** | secret 値を `cdk.json` や context に入れない。Secrets Manager や SSM Parameter Store を使う |
| **S3** | `blockPublicAccess`、暗号化、アクセスログ、ライフサイクルを用途に応じて設定する |
| **KMS** | 機密データは AWS managed key と customer managed key のどちらを使うか決める |
| **cdk-nag** | Well-Architected やセキュリティルールの自動チェックに使える |

---

## 論理 ID とリファクタリング

CloudFormation のリソースは論理 ID で追跡される。CDK では construct path から論理 ID が生成されるため、Construct ID や階層を変えると、同じ設定でもリソース置換になることがある。

```typescript
const bucket = new s3.Bucket(this, 'ReportsBucket');
```

既存リソースの置換を避けたい場合は、`cdk diff` で置換有無を確認する。どうしても論理 ID を固定したい場合は L1 construct に対して `overrideLogicalId` を使えるが、乱用すると後の保守が難しくなる。

```typescript
const cfnBucket = bucket.node.defaultChild as s3.CfnBucket;
cfnBucket.overrideLogicalId('ReportsBucket');
```

---

## Tips・ベストプラクティス

| カテゴリ | 内容 |
|----------|------|
| **命名** | Construct の ID はスタック内でユニークであればよい。物理リソース名は自動生成に任せると衝突しにくい |
| **環境分離** | `stage` コンテキストでリソース名を切り替える（`dev-MyBucket`, `prod-MyBucket`）か、スタック自体を分ける |
| **RemovalPolicy** | 本番では `RETAIN`、開発では `DESTROY` + `autoDeleteObjects: true` を使い分ける |
| **grant メソッド** | IAM 権限は `bucket.grantRead(fn)` のように grant メソッドを活用し、最小権限を保つ |
| **cdk.out** | `cdk synth` の出力は `.gitignore` に追加してよい（再生成できるため） |
| **Hotswap** | 一部リソースを高速デプロイ可能。CloudFormation を通さないため本番では使わない |
| **Escape hatch** | L2 で設定できないプロパティは `.node.defaultChild as CfnXxx` で L1 にアクセスして設定する |
| **cdk diff** | デプロイ前に必ず確認し、置換・削除・IAM 変更を重点的に見る |

```typescript
// Escape hatch の例
const bucket = new s3.Bucket(this, 'MyBucket');
const cfnBucket = bucket.node.defaultChild as s3.CfnBucket;
cfnBucket.addPropertyOverride('NotificationConfiguration.TopicConfigurations', [...]);
```

---

## 用語集

| 用語 | 説明 |
|------|------|
| **App** | CDK アプリのルート。すべての Stack の親 |
| **Stack** | CloudFormation スタックに対応する単位 |
| **Construct** | リソースの構成単位。再利用・ネスト可能 |
| **Synthesize（synth）** | CDK コードから CloudFormation テンプレートを生成すること |
| **Bootstrap** | CDK が使う S3・ECR などを AWS アカウントに事前作成すること |
| **cdk.out** | `cdk synth` で生成されるテンプレート出力ディレクトリ |
| **Token** | デプロイ時に確定する値のプレースホルダ（例: `${Token[Bucket.Arn.1]}`） |
| **Asset** | Lambda コードや Docker イメージなど、デプロイ時に S3/ECR へアップロードされる成果物 |
| **Context** | CDK app に渡す設定値や lookup 結果のキャッシュ |
| **Aspect** | スタック内の全コンストラクトを走査して横断的に変更を加える仕組み |
| **L1 / L2 / L3** | Construct の抽象レベル（L1=CF直接, L2=高レベル, L3=パターン） |
| **Grant メソッド** | IAM 権限をリソース側から付与するメソッド群（`grantRead`, `grantWrite` など） |
| **Hotswap** | CloudFormation を介さず Lambda/ECS を直接更新する高速デプロイモード |

---

## リンク集

- [AWS CDK 公式ドキュメント](https://docs.aws.amazon.com/cdk/v2/guide/)
- [API リファレンス（aws-cdk-lib）](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib-readme.html)
- [CDK Patterns（コミュニティのサンプル集）](https://cdkpatterns.com/)
- [Construct Hub（サードパーティ Construct の検索）](https://constructs.dev/)

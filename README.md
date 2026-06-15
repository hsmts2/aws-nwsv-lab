# aws-nwsv-lab

「基礎からのネットワーク&サーバー構築［改訂4版］」の学習内容を、AWS CloudFormation で再現するためのリポジトリです。

本リポジトリは、書籍で学習した AWS ネットワーク・サーバー構成を、自分の理解に基づいて Infrastructure as Code として整理したものです。書籍内容を転載するものではなく、学習した構成を CloudFormation テンプレートとして再現・管理することを目的としています。

---

<br>

## 概要

本リポジトリでは、書籍で構築する AWS ネットワーク・サーバー構成を CloudFormation テンプレートとして管理します。

手順ベースで作成した AWS リソースを IaC（Infrastructure as Code）として定義することで、構成の再現性、変更管理、レビュー、再デプロイを容易にすることを目的としています。

---

<br>

## 対象書籍と学習範囲

| 項目 | 内容 |
| --- | --- |
| 対象書籍 | 基礎からのネットワーク&サーバー構築［改訂4版］ |
| 学習テーマ | AWS 上でのネットワーク・サーバー構築 |
| 主な学習対象 | VPC / EC2 / Internet Gateway / NAT Gateway / Security Group / Apache / MariaDB / WordPress |
| 本リポジトリで作成する範囲 | VPC / EC2 / NAT Gateway / Security Group |
| 作成するAWS構成 | Web サーバー + DB サーバーの2層構成 |
| IaC | AWS CloudFormation |
| テンプレート形式 | YAML |
| リージョン | `ap-northeast-1` |

---

<br>

## リポジトリの目的

このリポジトリの目的は、単に CloudFormation テンプレートを保存することではなく、以下の観点を整理することです。

* AWS ネットワーク構成をコードとして再現する
* 手動構築した内容を CloudFormation に置き換える
* VPC、サブネット、ルートテーブル、セキュリティグループの関係を整理する
* Web サーバーと DB サーバーを分離した基本的な構成を理解する
* GitHub 上で学習成果をポートフォリオとして管理する

---

<br>

## アーキテクチャ

以下は、本リポジトリの CloudFormation テンプレートで作成する AWS 構成図です。

<img src="./diagrams/aws-nwsv4-architecture.png" alt="Architecture" width="900">

---

<br>

## 作成される構成

```text
Internet
  |
  | HTTP:80
  v
Web Server EC2
  |
  | TCP:3306
  v
DB Server EC2
```

ネットワーク構成は以下です。

```text
VPC: 10.0.0.0/16
├── Public Subnet: 10.0.1.0/24
│   ├── NAT Gateway
│   └── Web Server
│
└── Private Subnet: 10.0.2.0/24
    └── DB Server
```

---

<br>

## 作成される主な AWS リソース

| Resource | 設定値 / 備考 |
| --- | --- |
| VPC | CIDR: `10.0.0.0/16` / DNS ホスト名: 有効 |
| パブリックサブネット | `10.0.1.0/24` |
| プライベートサブネット | `10.0.2.0/24` |
| Internet Gateway | VPC にアタッチ |
| NAT Gateway | パブリックサブネットに配置 / Elastic IP を割り当て |
| ルートテーブル | パブリック用・プライベート用 |
| セキュリティグループ | Web 用 / DB 用 |
| IAM ロール | EC2 用 IAM ロール / SSM Session Manager 対応 |
| EC2 Web サーバー | Amazon Linux 2023 |
| EC2 DB サーバー | Amazon Linux 2023 |

---

<br>

## 本テンプレートで対象外にしたもの

| 対象外 | 理由 |
| --- | --- |
| ALB | まずは Web / DB の2層構成に絞るため |
| RDS | 書籍の学習構成に合わせ、EC2 上の DB サーバーとして構築するため |
| Route 53 | ドメイン取得やDNS設定が環境依存になるため |
| ACM | HTTPS化にはドメインと証明書検証が必要になるため |

---

<br>

## ディレクトリ構成

```text
.
├── README.md
├── templates/
│   └── aws-nwsv4.yaml
├── parameters/
│   └── aws-nwsv4-lab.example.json
└── diagrams/
    └── aws-nwsv4-architecture.png
```

---

<br>

## CloudFormation テンプレート

CloudFormation テンプレートは以下に格納しています。

[templates/aws-nwsv4.yaml](templates/aws-nwsv4.yaml)

このテンプレートでは、VPC、サブネット、Internet Gateway、NAT Gateway、ルートテーブル、セキュリティグループ、IAM ロール、EC2 インスタンスなどをまとめて作成します。

---

<br>

## パラメータファイル

パラメータファイルのサンプルは以下に格納しています。

[parameters/aws-nwsv4-lab.example.json](parameters/aws-nwsv4-lab.example.json)

実際にデプロイする場合は、サンプルファイルをコピーして使用します。

```bash
cp parameters/aws-nwsv4-lab.example.json parameters/aws-nwsv4-lab.json
```

`parameters/aws-nwsv4-lab.json` には、自分の環境に合わせた値を設定します。実環境用の値を含むため、GitHub にはコミットしない運用とします。

---

<br>

## 前提条件

以下の環境が必要です。

* AWS CLI v2
* AWS CLI の認証設定済み環境
* デプロイ先 AWS アカウント
* デプロイ先リージョン: `ap-northeast-1`
* 既存の EC2 キーペア
* CloudFormation、EC2、VPC、IAM 関連リソースを作成・更新・削除できる IAM 権限

---

<br>

## デプロイ・運用方法

### パラメータ準備

```bash
cp parameters/aws-nwsv4-lab.example.json parameters/aws-nwsv4-lab.json
```

### テンプレート検証

```bash
aws cloudformation validate-template \
  --template-body file://templates/aws-nwsv4.yaml
```

### スタック作成

```bash
aws cloudformation create-stack \
  --stack-name aws-nwsv4-lab \
  --template-body file://templates/aws-nwsv4.yaml \
  --parameters file://parameters/aws-nwsv4-lab.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ap-northeast-1
```

### スタック状態確認

```bash
aws cloudformation describe-stacks \
  --stack-name aws-nwsv4-lab \
  --region ap-northeast-1
```

### 動作確認

スタック作成後、CloudFormation の Outputs や EC2 インスタンスの状態を確認し、想定した Web / DB 構成が作成されていることを確認します。

### スタック更新

```bash
aws cloudformation update-stack \
  --stack-name aws-nwsv4-lab \
  --template-body file://templates/aws-nwsv4.yaml \
  --parameters file://parameters/aws-nwsv4-lab.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ap-northeast-1
```

### スタック削除

```bash
aws cloudformation delete-stack \
  --stack-name aws-nwsv4-lab \
  --region ap-northeast-1
```

---

<br>

## 設計上の補足

このリポジトリでは、学習対象を Web / DB の2層構成に絞っています。ALB、RDS、Route 53、ACM などは、より発展的な構成を扱う別リポジトリで整理します。

---

<br>

## 書籍との差分

書籍で手動構築した内容をもとに、CloudFormation で同等の学習環境を再現できるようにしています。リソース名やパラメータ値は、再利用しやすいようにテンプレート側で整理しています。

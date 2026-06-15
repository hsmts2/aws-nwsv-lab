# aws-nwsv-lab

「基礎からのネットワーク&サーバー構築［改訂4版］」の学習内容を、AWS CloudFormation で再現するためのリポジトリです。

本リポジトリは、書籍で学習した AWS ネットワーク・サーバー構成を、自分の理解に基づいて Infrastructure as Code として整理したものです。
書籍内容を転載するものではなく、学習した構成を CloudFormation テンプレートとして再現・管理することを目的としています。

---

<br>

## 概要

本リポジトリでは、書籍で構築する AWS ネットワーク・サーバー構成を CloudFormation テンプレートとして管理します。

手順ベースで作成した AWS リソースを IaC（Infrastructure as Code）として定義することで、構成の再現性、変更管理、レビュー、再デプロイを容易にすることを目的としています。

> **学習目的**: VPC 設計、パブリックサブネット、プライベートサブネット、Internet Gateway、NAT Gateway、セキュリティグループ、EC2、Web / DB 分離構成の理解

---

<br>

## 対象書籍と学習範囲

| 項目        | 内容                                                                                         |
| --------- | ------------------------------------------------------------------------------------------ |
| 対象書籍      | 基礎からのネットワーク&サーバー構築［改訂4版］                                                                   |
| 学習テーマ     | AWS 上でのネットワーク・サーバー構築                                                                       |
| 主な学習対象    | VPC / EC2 / Internet Gateway / NAT Gateway / Security Group / Apache / MariaDB / WordPress |
| 作成するAWS構成 | Web サーバー + DB サーバーの2層構成                                                                    |
| IaC       | AWS CloudFormation                                                                         |
| テンプレート形式  | YAML                                                                                       |
| リージョン     | `ap-northeast-1`                                                                           |

このリポジトリでは、書籍で手動構築した内容をもとに、CloudFormation で同等の学習環境を再現できるようにしています。

---

<br>

## リポジトリの目的

このリポジトリの目的は、単に CloudFormation テンプレートを保存することではなく、以下の観点を整理することです。

* AWS ネットワーク構成をコードとして再現する
* 手動構築した内容を CloudFormation に置き換える
* VPC、サブネット、ルートテーブル、セキュリティグループの関係を整理する
* Web サーバーと DB サーバーを分離した基本的な構成を理解する
* GitHub 上で学習成果をポートフォリオとして管理する
* README、構成図、パラメータファイル、`.gitignore` を含めたリポジトリ運用を学ぶ

---

<br>

## このリポジトリで示せるスキル

* AWS VPC / Subnet / Route Table / Security Group の基本設計
* CloudFormation によるインフラ構成のコード化
* パブリックサブネットとプライベートサブネットを分けた2層構成の理解
* Web サーバーと DB サーバーを分離した基本構成の整理
* 秘匿情報を Git 管理から除外する運用
* 構成図、README、パラメータ例を含めた再現可能なドキュメント整備

---

<br>

## アーキテクチャ

以下は、本リポジトリの CloudFormation テンプレートで作成する AWS 構成図です。

<img src="./diagrams/aws-nwsv4-architecture.png" alt="Architecture" width="900">

---

<br>

## 作成される主な AWS リソース

| Resource         | 設定値 / 備考                               |
| ---------------- | -------------------------------------- |
| VPC              | CIDR: `10.0.0.0/16` / DNS ホスト名: 有効     |
| パブリックサブネット       | `10.0.1.0/24`                          |
| プライベートサブネット      | `10.0.2.0/24`                          |
| Internet Gateway | VPC にアタッチ                              |
| NAT Gateway      | パブリックサブネットに配置 / Elastic IP を割り当て       |
| ルートテーブル          | パブリック用・プライベート用                         |
| セキュリティグループ       | Web 用 / DB 用                           |
| IAM ロール          | EC2 用 IAM ロール / SSM Session Manager 対応 |
| EC2 Web サーバー     | Amazon Linux 2023                      |
| EC2 DB サーバー      | Amazon Linux 2023                      |

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
├── diagrams/
│   └── aws-nwsv4-architecture.png
└── docs/
    └── notes.md
```

---

<br>

## CloudFormation テンプレート

CloudFormation テンプレートは以下に格納しています。

```text
templates/aws-nwsv4.yaml
```

このテンプレートでは、VPC、サブネット、Internet Gateway、NAT Gateway、ルートテーブル、セキュリティグループ、IAM ロール、EC2 インスタンスなどをまとめて作成します。

---

<br>

## パラメータファイル

パラメータファイルのサンプルは以下に格納しています。

```text
parameters/aws-nwsv4-lab.example.json
```

実際にデプロイする場合は、サンプルファイルをコピーして使用します。

```bash
cp parameters/aws-nwsv4-lab.example.json parameters/aws-nwsv4-lab.json
```

`parameters/aws-nwsv4-lab.json` には、自分の環境に合わせた値を設定します。

例：

```json
[
  {
    "ParameterKey": "KeyName",
    "ParameterValue": "your-key-name"
  },
  {
    "ParameterKey": "MyIpCidr",
    "ParameterValue": "your-global-ip/32"
  }
]
```

> `parameters/aws-nwsv4-lab.json` は実環境用のファイルのため、GitHub にはコミットしない運用とします。

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

## 使い方

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

## セキュリティ

以下の情報は絶対にコミットしないでください。

| 禁止ファイル / 情報                   | 理由                        |
| ----------------------------- | ------------------------- |
| `*.pem`                       | EC2 接続用秘密鍵                |
| `*.key`                       | 秘密鍵ファイル                   |
| `*accesskey*` / `*secretkey*` | AWS 認証情報                  |
| `.env`                        | 環境変数ファイル                  |
| 実環境用パラメータファイル                 | IP アドレスや環境固有情報を含む可能性があるため |
| DB パスワード                      | 認証情報漏洩防止のため               |

これらは `.gitignore` で除外することを推奨します。

万一、AWS アクセスキーや秘密鍵をコミットした場合は、該当キーを即座に無効化・ローテーションしてください。

---

<br>

## 課金に関する注意

このテンプレートでは、以下のリソースで課金が発生する可能性があります。

| リソース | 注意点 |
| --- | --- |
| EC2 | インスタンス稼働時間に応じて課金 |
| EBS | ボリューム容量に応じて課金 |
| NAT Gateway | 時間課金とデータ処理課金が発生 |
| Elastic IP | NAT Gateway で使用 |
| データ転送 | 通信量に応じて課金される場合あり |

特に NAT Gateway は、起動している時間に応じて課金されるため、検証後は必ずスタックを削除してください。

---

<br>

## 補足メモ

学習メモや補足手順は、以下にまとめます。

```text
docs/notes.md
```

---

<br>

## 今後の拡張案

今後、以下の拡張も検討できます。

* ALB による Web サーバー冗長化
* EC2 上の MariaDB から RDS への置き換え
* CloudWatch Logs によるログ収集
* Systems Manager Session Manager による SSH レス運用
* GitHub Actions による CloudFormation Lint / Validate

---

<br>

## 参考

* 基礎からのネットワーク&サーバー構築［改訂4版］
* [AWS CloudFormation ユーザーガイド](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/)
* [Amazon VPC ユーザーガイド](https://docs.aws.amazon.com/ja_jp/vpc/latest/userguide/)
* [AWS CLI コマンドリファレンス](https://docs.aws.amazon.com/cli/latest/reference/)

---
slug: /ja/manage/security/cloud-endpoints-api
sidebar_label: クラウドIPアドレス
title: クラウドIPアドレス
---

## 静的IPの一覧

以下の表は、ClickHouse Cloudでサポートされている各クラウドとリージョンにおける静的IPとS3エンドポイントを示しています。

### AWS

#### Egress IPs

| リージョン | IPアドレス |
|--------|------|
| ap-northeast-1 | `35.73.179.23` `54.248.225.163` `54.65.53.160` |
| ap-south-1 | `15.206.7.77` `3.110.39.68` `3.6.83.17` |
| ap-southeast-1 | `46.137.240.41` `52.74.24.166` `54.254.37.170` |
| ap-southeast-2 | `13.210.79.90` `13.236.190.252` `13.54.63.56` |
| eu-central-1 | `18.197.49.136` `3.64.109.93` `3.74.177.59` |
| eu-west-1 | `108.128.86.193` `34.240.176.195` `54.73.98.215` |
| eu-west-2 | `13.43.228.235` `18.135.36.159` `3.10.239.194` |
| us-east-1 | `18.211.40.49` `35.175.32.241` `44.197.47.227` `44.208.152.165` `52.205.46.187` `52.22.199.32` |
| us-east-2 | `18.117.209.120` `3.135.147.1` `3.21.42.89` |
| us-west-2 | `35.165.97.55` `44.236.63.111` `54.244.160.153` |

#### Ingress IPs

| リージョン | IPアドレス |
|--------|------|
| ap-northeast-1 | `18.182.162.251` `52.194.58.178` `54.150.231.136` |
| ap-south-1 | `15.206.78.111` `3.6.185.108` `43.204.6.248` |
| ap-southeast-1 | `18.138.54.172` `18.143.38.5` `18.143.51.125` |
| ap-southeast-2 | `3.105.241.252` `3.24.14.253` `3.25.31.112` |
| eu-central-1 | `3.125.141.249` `3.75.55.98` `52.58.240.109` |
| eu-west-1 | `79.125.122.80` `99.80.3.151` `99.81.5.155` |
| eu-west-2 | `18.133.60.223` `18.134.36.146` `18.170.151.77` |
| us-east-1 | `3.224.78.251` `3.217.93.62` `52.4.220.199` `54.166.56.105` `44.210.169.212` `52.206.111.15` |
| us-east-2 | `18.216.18.121` `18.218.245.169` `18.225.29.123` |
| us-west-2 | `35.82.252.60` `35.85.205.122` `44.226.232.172` |

#### S3エンドポイント

| リージョン | IPアドレス |
|--------|------|
| ap-northeast-1 | `vpce-0047c645aecb997e7` |
| ap-south-1 | `vpce-0a975c9130d07276d` |
| ap-southeast-1 | `vpce-04c0b7c7066498854` |
| ap-southeast-2 | `vpce-0b45293a83527b13c` |
| eu-central-1 | `vpce-0c58e8f7ed0f63623` |
| eu-west-1 | `vpce-0c85c2795779d8fb2` |
| eu-west-2 | `vpce-0ecd990a9f380d784` |
| us-east-1 | `vpce-05f1eeb392b983932` `vpce-0b8b558ea42181cf6` |
| us-east-2 | `vpce-09ff616fa76e09734` |
| us-west-2 | `vpce-0bc78cc5e63dfb27c` |

### GCP

#### Egress IPs

| リージョン | IPアドレス |
|--------|------|
| europe-west4 | `34.147.18.130` `34.90.110.137` `34.90.16.52` `34.91.142.156` `35.234.163.128` |
| asia-southeast1 | `34.124.237.2` `34.142.232.74` `34.143.238.252` `35.240.251.145` `35.247.141.182` |
| us-central1 | `34.136.25.254` `34.170.139.51` `34.172.174.233` `34.173.64.62` `34.66.234.85` |
| us-east1 | `104.196.202.44` `34.74.136.232` `104.196.211.50` `34.74.115.64` `35.237.218.133` |

#### Ingress IPs

| リージョン | IPアドレス |
|--------|------|
| europe-west4 | `35.201.102.65` |
| asia-southeast1 | `34.160.80.214` |
| us-central1 | `35.186.193.237` |
| us-east1 | `34.36.105.62` |

### Azure

#### Egress IPs

| リージョン | IPアドレス |
|--------|------|
| eastus2 | `20.109.60.147` `20.242.123.110` `20.75.77.143` |
| westus3 | `20.14.94.21` `20.150.217.205` `20.38.32.164` |
| germanywestcentral | `20.218.133.244` `20.79.167.238` `51.116.96.61` |

### Ingress IPs

| リージョン | IPアドレス |
|--------|------|
| eastus2 | `4.152.12.124` |
| westus3 | `4.227.34.126` |
| germanywestcentral | `4.182.8.168` |

## 静的IPのAPI

静的IPのリストをプログラムで取得する必要がある場合は、以下のClickHouse Cloud APIエンドポイントを使用できます: [`https://api.clickhouse.cloud/static-ips.json`](https://api.clickhouse.cloud/static-ips.json)。このAPIは、リージョンごとおよびクラウドごとのingress/egress IPやS3エンドポイントの情報を提供します。

MySQLやPostgreSQLエンジンのような統合を使用している場合、ClickHouse Cloudがインスタンスにアクセスできるように承認する必要があるかもしれません。このAPIを使用して、パブリックIPを取得し、GCPの`firewalls`または`Authorized networks`、AzureやAWSの`Security Groups`、もしくは他の任意のインフラeコネクトストの管理システムで設定することができます。

例えば、`ap-south-1`リージョンでホストされているAWSのClickHouse Cloudサービスからのアクセスを許可するには、そのリージョンの`egress_ips`アドレスを追加します:

```
❯ curl -s https://api.clickhouse.cloud/static-ips.json | jq '.'
{
  "aws": [
    {
      "egress_ips": [
        "3.110.39.68",
        "15.206.7.77",
        "3.6.83.17"
      ],
      "ingress_ips": [
        "15.206.78.111",
        "3.6.185.108",
        "43.204.6.248"
      ],
      "region": "ap-south-1",
      "s3_endpoints": "vpce-0a975c9130d07276d"
    },
...
```

例えば、`us-east-2`で稼働しているAWS RDSインスタンスがClickHouse Cloudサービスに接続する必要がある場合、以下のInboundセキュリティグループルールを設定します:

![AWS Security group rules](@site/docs/ja/_snippets/images/aws-rds-mysql.png)

同じく`us-east-2`のClickHouse CloudサービスがGCP上のMySQLに接続する場合、`Authorized networks`は次のようになります:

![GCP Authorized networks](@site/docs/ja/_snippets/images/gcp-authorized-network.png)
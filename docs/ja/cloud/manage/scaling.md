---
sidebar_position: 1
sidebar_label: 自動スケーリング
slug: /ja/manage/scaling
description: ClickHouse Cloudの自動スケーリング設定
keywords: [autoscaling, auto scaling, scaling, horizontal, vertical, bursts]
---

# 自動スケーリング

スケーリングとは、クライアントの需要に応じて使用可能なリソースを調整する能力のことです。サービスはAPIをプログラム的に呼び出したり、UIでシステムリソースを調整する設定を変更することで手動でスケーリングできます。代わりに、サービスはアプリケーションの要求に合わせて**自動スケーリング**することも可能です。

:::note
スケーリングは**Production**ティアのサービスにのみ適用されます。**Development**ティアのサービスはスケーリングしません。スケーリングするためには、**Development**ティアから**Production**ティアにサービスを**アップグレード**する必要があります。一度**Development**サービスがアップグレードされると、ダウングレードすることはできません。
:::

## ClickHouse Cloudにおけるスケーリングの仕組み
**Production**サービスは、（より大きなレプリカに切り替えることで）垂直にスケーリングしたり、（同じサイズのレプリカを追加することで）水平にスケーリングしたりできます。デフォルトでは、ClickHouse Cloudの**Production**サービスは3つの異なる可用性ゾーンに3つのレプリカで運用されます。垂直スケーリングは、長時間の挿入や読み取りに多くのメモリを必要とするクエリに役立ち、水平スケーリングは並列化によって同時実行クエリをサポートするのに役立ちます。

現在、ClickHouse Cloudはサービスを垂直にのみ自動スケーリングします。水平にスケーリングするには（現在プライベートプレビュー中）、ClickHouse CloudコンソールまたはCloud APIを使用する必要があります。サービスで水平スケーリングを有効にするには、support@clickhouse.comにお問い合わせのうえ、[セルフサーブ水平スケーリング](#self-serve-horizontal-scaling)のセクションをご覧ください。

### 垂直自動スケーリング
**Production**サービスは、CPUとメモリの使用量に基づいて自動スケーリングされます。過去30時間のルックバックウィンドウにわたってサービスの使用履歴を常に監視し、スケーリングの判断を行います。使用量が特定の閾値を上回ったり下回ったりした場合、需要に応じてサービスを適切にスケーリングします。

CPUベースの自動スケーリングは、CPU使用率が50-75％の範囲の上限閾値を超えたときに始動します。この時点で、クラスタへのCPU割り当てが倍増されます。CPU使用率が上限閾値の半分を下回った場合（たとえば、上限閾値が50％の場合、25％）、CPU割り当てが半減されます。

メモリベースの自動スケーリングでは、クラスタは最大メモリ使用量の125％、またはメモリ不足エラー（OOM）が発生した場合は最大150％までスケーリングされます。

CPUまたはメモリの推奨値で**より大きい方**が選ばれ、サービスに割り当てられるCPUとメモリは`1` CPUと`4 GiB`メモリの同期インクリメントでスケーリングされます。

注意: 現在の実装では、垂直自動スケーリングはメモリとCPUのニーズが緩やかに増加する場面ではうまく機能し、保守的になる傾向があります。ワークロードの急増に対応できるよう、より動的にし、スケーリングのためにより積極的なCPU/メモリ閾値を使用し、適切なルックバックウィンドウを使用して垂直スケーリングの意思決定を行うよう改善を進めています。

### 垂直自動スケーリングの設定
ClickHouse Cloud Productionサービスのスケーリングは、**Admin**ロールを持つ組織のメンバーによって調整できます。垂直自動スケーリングを設定するには、サービス詳細ページの**設定**タブに移動し、最小および最大メモリ、及びCPU設定を調整します。

<img alt="スケーリング設定ページ" style={{width: '450px', marginLeft: 0}} src={require('./images/AutoScaling.png').default} />

**最大メモリ**を**最小メモリ**よりも高く設定してください。すると、必要に応じてその範囲内でサービスがスケールします。これらの設定は、初期のサービス作成フローでも利用可能です。サービス内の各レプリカには、同じメモリとCPUリソースが割り当てられます。

これらの値を同じに設定することも選択でき、その場合、特定の設定にサービスを固定します。そうすることで、選択したサイズにすぐにスケーリングが強制されます。これは、クラスタでの任意の自動スケーリングを無効にし、この設定を超えたCPUまたはメモリ使用量の増加からサービスを保護しなくなることに注意してください。

## セルフサービスによる水平スケーリング {#self-serve-horizontal-scaling}

ClickHouse Cloudの水平スケーリングは現在**プライベートプレビュー**中です。一旦水平スケーリングがサービスで有効になると、ClickHouse Cloud [公開API](https://clickhouse.com/docs/ja/cloud/manage/api/swagger#/paths/~1v1~1organizations~1:organizationId~1services~1:serviceId~1scaling/patch)を使用して、サービスのスケーリング設定を更新することでサービスをスケールできます。

- 機能がサービスで有効になっていない場合、リクエストは`BAD_REQUEST: Adjusting number of replicas is not enabled for your instance"というエラーで拒否されます。このエラーが表示され、スケーリングが既に有効になっていると考える場合は、ClickHouse Cloudサポートにお問い合わせください。
- **Production** ClickHouseサービスは最低でも`3`つのレプリカが必要です。現在、**Production**サービスがスケールアウト可能な最大レプリカ数は`20`です。これらの制限は時間とともに増加します。現在、より高い制限が必要な場合は、ClickHouse Cloudサポートチームにご連絡ください。
- 現在、スケールイン操作中に削除されるレプリカのシステムテーブルデータは保持されません。これは、システムテーブルデータを利用しているダッシュボードや他の機能に影響を与える可能性があります。

### APIを介した水平スケーリング

クラスタを水平スケーリングするには、APIを通じて`PATCH`リクエストを発行し、レプリカ数を調整します。以下のスクリーンショットは、`3`レプリカのクラスタを`6`レプリカにスケールアウトするAPIコールと、その対応するレスポンスを示しています。

<img alt="スケーリング設定ページ"
    style={{width: '500px', marginLeft: 0}}
    src={require('./images/scaling-patch-request.png').default} />

*レプリカ数を更新する`PATCH`リクエスト*

<img alt="スケーリング設定ページ"
    style={{width: '450px', marginLeft: 0}}
    src={require('./images/scaling-patch-response.png').default} />

*`PATCH`リクエストからのレスポンス*

新しいスケーリングリクエストや複数のリクエストを連続して発行すると、そのうち1つが進行中の場合、スケーリングサービスは中間状態を無視し、最終的なレプリカ数に収束します。

### UIを介した水平スケーリング

UIからサービスを水平スケーリングするには、**設定**ページでサービスのノード数を調整します。

<img alt="スケーリング設定ページ"
    style={{width: '500px', marginLeft: 0}}
    src={require('./images/scaling-configure.png').default} />

*ClickHouse Cloudコンソールからのサービススケーリング設定*

サービスのスケールが完了すると、クラウドコンソールのメトリックダッシュボードに正しい割り当てが表示されるはずです。以下のスクリーンショットは、クラスタが`96 GiB`の総メモリにスケールしていることを示しており、これは`6`レプリカ、各レプリカにGiBのメモリが割り当てられています。

<img alt="スケーリング設定ページ"
    style={{width: '500px', marginLeft: 0}}
    src={require('./images/scaling-memory-allocation.png').default} />

## 自動アイドリング
**設定**ページでは、サービスが非アクティブ時に自動アイドリングが許可されるかどうかを選択できます（すなわち、サービスがユーザー提出のクエリを実行していないとき）。自動アイドリングにより、サービスのコストを削減でき、サービスが一時停止中はコンピューティングリソースに対して請求が発生しません。

:::danger 自動アイドリングを使用しない場合
自動アイドリングは、クエリに応答する前に遅延を許容できる場合のみ使用してください。サービスが一時停止されると、その間の接続がタイムアウトします。自動アイドリングは、使用頻度が少なく、遅延が許容できるサービスに理想的です。頻繁に使用される顧客向け機能を動かすサービスには推奨されません。
:::

## 突発的なワークロードの取り扱い
予想されるワークロードのスパイクがある場合は、[ClickHouse Cloud API](/docs/ja/cloud/manage/api/services-api-reference.md)を使用して、あらかじめサービスを拡張し、スパイクを処理した後に需要が落ち着いたらスケールダウンすることができます。各レプリカの現在のCPUコアとメモリリソースを理解するために、以下のクエリを実行できます：

```sql
SELECT *
FROM clusterAllReplicas('default', view(
    SELECT
        hostname() AS server,
        anyIf(value, metric = 'CGroupMaxCPU') AS cpu_cores,
        formatReadableSize(anyIf(value, metric = 'CGroupMemoryTotal')) AS memory
    FROM system.asynchronous_metrics
))
ORDER BY server ASC
SETTINGS skip_unavailable_shards = 1
```
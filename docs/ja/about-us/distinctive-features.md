---
slug: /ja/about-us/distinctive-features
sidebar_label: ClickHouseはなぜユニークなのか？
sidebar_position: 50
description: 他のデータベース管理システムとは異なるClickHouseの特徴を理解する
---

# ClickHouseの特徴

## 真の列指向データベース管理システム

真の列指向DBMSでは、値に余分なデータが保存されません。これは、値の長さをその隣に「数値」として保存することを避けるために、固定長の値をサポートする必要があることを意味します。たとえば、10億個のUInt8型の値は圧縮されていないときに約1GBを消費するべきであり、これはCPUの使用に強く影響します。データを圧縮されずに（「ゴミ」がなく）コンパクトに保存することが重要で、というのも、データの解凍速度（CPU使用量）は圧縮されていないデータのボリュームに主に依存するからです。

これは、異なるカラムの値を別々に保存できるが、分析クエリを効果的に処理できないシステム、例えばHBase、BigTable、Cassandra、HyperTableなどとは対照的です。これらのシステムでは、1秒間に約10万行のスループットは得られますが、数億行は得られません。

最終的に、ClickHouseはデータベース管理システムであり、単一のデータベースではありません。実行時にテーブルやデータベースを作成し、データをロードし、サーバーを再構成せずにクエリを実行できます。

## データ圧縮 {#data-compression}

一部の列指向DBMSはデータ圧縮を使用しません。しかし、データ圧縮は優れたパフォーマンスを達成するための重要な要素です。

ディスクスペースとCPU消費の間のトレードオフを考慮した効率的な汎用圧縮コーデックに加え、ClickHouseは特定のデータ型に特化した[コーデック](/docs/ja/sql-reference/statements/create/table.md#specialized-codecs)を提供し、ClickHouseはタイムシリーズ型のような特定の用途に優れたデータベースと競争し、さらに優れます。

## データのディスクストレージ {#disk-storage-of-data}

データを主キーで物理的にソートして保持すると、特定の値または値の範囲に基づいてデータを数十ミリ秒未満の低遅延で抽出することが可能になります。SAP HANAやGoogle PowerDrillのような一部の列指向DBMSはRAMでのみ動作します。このアプローチではリアルタイム分析に必要以上のハードウェア予算を確保する必要があります。

ClickHouseは通常のハードドライブで動作するように設計されており、これはデータストレージあたりのGB単位のコストが低いことを意味しますが、利用可能な場合はSSDや追加のRAMも完全に使用します。

## 複数コアでの並列処理 {#parallel-processing-on-multiple-cores}

大きなクエリは自然に並列化され、現在のサーバーで利用可能なすべてのリソースを利用します。

## 複数サーバーでの分散処理 {#distributed-processing-on-multiple-servers}

上記の列指向DBMSのほとんどに分散クエリ処理のサポートはありません。

ClickHouseでは、データは異なるシャードに存在できます。各シャードは耐障害性のために使用されるレプリカのグループであることができます。すべてのシャードは、ユーザーにとって透明に並列でクエリを実行するために使用されます。

## SQLサポート {#sql-support}

ClickHouseは、ANSI SQL標準とほぼ互換性のある[SQL言語](/ja/sql-reference/)をサポートしています。

サポートされているクエリには、[GROUP BY](../sql-reference/statements/select/group-by.md)、[ORDER BY](../sql-reference/statements/select/order-by.md)、[FROM](../sql-reference/statements/select/from.md)でのサブクエリ、[JOIN](../sql-reference/statements/select/join.md)句、[IN](../sql-reference/operators/in.md)演算子、[ウィンドウ関数](../sql-reference/window-functions/index.md)やスカラーサブクエリが含まれます。

相関（依存）サブクエリは執筆時点ではサポートされていませんが、将来的には利用可能になる可能性があります。

## ベクター計算エンジン {#vector-engine}

データはカラムで保存されるだけでなく、ベクター（カラムの一部）によって処理され、高いCPU効率が達成されます。

## リアルタイムデータ挿入 {#real-time-data-updates}

ClickHouseは主キーを持つテーブルをサポートしています。主キーの範囲でクエリを迅速に実行するために、データはインクリメンタルにMergeTreeを使用してソートされます。これにより、データを継続的にテーブルに追加することができます。新しいデータが取り込まれた際にロックは取得されません。

## 主インデックス {#primary-index}

データが主キーで物理的にソートされていることで、特定の値または値の範囲に基づいてデータを数十ミリ秒未満の低遅延で抽出することが可能になります。

## 二次インデックス {#secondary-indexes}

他のデータベース管理システムとは異なり、ClickHouseの二次インデックスは特定の行や行範囲を指しません。代わりに、クエリのフィルタリング条件に一致しないすべての行があるデータ部分でそれを読み込まないようにデータベースに事前に知らせるため、[データスキッピングインデックス](../engines/table-engines/mergetree-family/mergetree.md#table_engine-mergetree-data_skipping-indexes)と呼ばれます。

## オンラインクエリへの適合性 {#suitable-for-online-queries}

ほとんどのOLAPデータベース管理システムは、1秒未満のレイテンシーでのオンラインクエリを目指していません。代替システムでは、数十秒から数分間のレポート生成時間が許容されることが多く、時にはさらに時間がかかり、オフラインでレポートを準備することを強いられることがあります（事前に、または「後で来てください」という形で）。

ClickHouseでは、「低遅延」とは、ユーザーインターフェースページが読み込まれるのと同時に、遅延なく、事前に回答を準備しようとせずにクエリを処理できることを意味します。言い換えれば、オンラインで。

## 近似計算のサポート {#support-for-approximated-calculations}

ClickHouseは、性能を犠牲にして精度をトレードオフする様々な方法を提供しています：

1.  異なる値の数、中央値、および分位数を近似計算するための集計関数。
2.  データの一部（サンプル）に基づいてクエリを実行し、近似結果を得る。この場合、ディスクから取得されるデータの量は比例して少なくなります。
3.  すべてのキーではなく、ランダムなキーの制限された数の集計を実行する。データ内のキー分布の条件によっては、少ないリソースで合理的に正確な結果を提供します。

## アダプティブジョインアルゴリズム {#adaptive-join-algorithm}

ClickHouseは、複数のテーブルを[JOIN](../sql-reference/statements/select/join.md)する際に、主にハッシュジョインアルゴリズムを選択し、複数の大きなテーブルがある場合にはマージジョインアルゴリズムにフォールバックします。

## データレプリケーションとデータ整合性のサポート {#data-replication-and-data-integrity-support}

ClickHouseは非同期マルチマスターレプリケーションを使用します。利用可能なレプリカに書き込まれた後、残りのすべてのレプリカがバックグラウンドでそのコピーを取得します。システムは異なるレプリカに同一のデータを保持します。ほとんどの障害からの回復は自動または複雑な場合には半自動で行われます。

詳細については、[データレプリケーション](../engines/table-engines/mergetree-family/replication.md)のセクションを参照してください。

## ロールベースのアクセス制御 {#role-based-access-control}

ClickHouseはSQLクエリを使用したユーザーアカウント管理を実装し、ANSI SQL標準および一般的なリレーショナルデータベース管理システムで見られる[ロールベースのアクセス制御の構成](/docs/ja/guides/sre/user-management/index.md)を可能にします。

## 欠点と考えられる機能 {#clickhouse-features-that-can-be-considered-disadvantages}

1.  本格的なトランザクションなし。
2.  高速かつ低遅延で既に挿入されたデータを修正または削除する能力の欠如。データを整理したり変更したりするためにバッチ削除や更新は利用可能ですが、例えば[GDPR](https://gdpr-info.eu)に準拠するために別の方法があります。
3.  スパースなインデックスがClickHouseをポイントクエリでシングル行をキーによって取得するのにあまり効率的ではないようにします。
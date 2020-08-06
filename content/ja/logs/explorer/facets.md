---
title: ログファセット
kind: documentation
description: ログファセットとファセットパネル
aliases:
  - /ja/logs/facets
further_reading:
  - link: logs/explorer/analytics
    tag: ドキュメント
    text: ログ分析の実行
  - link: logs/explorer/patterns
    tag: ドキュメント
    text: ログ内のパターン検出
  - link: logs/processing
    tag: ドキュメント
    text: ログの処理方法
  - link: logs/explorer/saved_views
    tag: ドキュメント
    text: ログエクスプローラーの自動構成
---
{{< img src="logs/explorer/facet/facets_in_explorer.gif" alt="エクスプローラーファセットのファセット" style="width:100%;">}}

## 概要

ファセットは、インデックス付きログのユーザー定義のタグと属性です。これらは、定性的または定量的なデータ分析を目的としています。そのため、ログエクスプローラーでこれらを使用して、次のことができます。

- [ログを検索する][1]
- [ログパターンを定義する][2]
- [ログ分析を実行する][3]

ファセットを使用すると、[ログモニター][4]のログ、[ダッシュボード][5]のログウィジェット、[ノートブック][6]を操作することもできます。

**注**: [ログ処理][7]、[ライブテイル検索][8]、[アーカイブ][9]転送、リハイドレート、またはログからの[メトリクス生成][10]をサポートするためのファセットは必要ありません。また、フィルターを使用して[パイプライン][11]および[インデックス][12]にログをルーティングするためのファセットや、[除外フィルター][13]を使用してインデックスからログを除外またはサンプリングするためのファセットも必要ありません。これらすべてのコンテキストで、オートコンプリート機能は既存のファセットに依存しますが、入力ログに一致する入力はすべて機能します。

### 定性的ファセット: ディメンション

次が必要な場合は、定性的ファセットを使用します。

- 特定の値に対してログを**フィルタリング**する。たとえば、`environment` [タグ][14]のファセットを作成して、トラブルシューティングを開発、ステージング、または本番環境にまで絞り込みます。
- 値の**相対的なインサイトを取得**する。たとえば、`http.network.client.geoip.country.iso_code` のファセットを作成して、[NGINX][15] ウェブアクセスログの 5XX エラーの数ごとに最も影響を受ける上位の国を確認し、Datadog [GeoIP Processor][16] で加工します。
- **一意の値をカウント**する。たとえば、[Kong][17] ログから `user.email` のファセットを作成して、ウェブサイトに毎日接続しているユーザー数を把握します。

#### 型 {#types-qualitative-facets}

定性的ファセットには、文字列または数値（整数）型を指定できます。文字列型をディメンションに割り当てるとすべてのケースで機能しますが、ディメンションで整数型を使用すると、前述のすべての機能に加えて範囲フィルタリングが有効になります。たとえば、`http.status_code:[200 TO 299]` は整数型のディメンションで使用する有効なクエリです。参考として[検索構文][1]を参照してください。

### 定量的ファセット: メジャー

次が必要な場合は、メジャーを使用します。

- 複数のログから**値を集計**する。たとえば、マップサーバーの [Varnish キャッシュ][18]が提供するタイルのサイズのメジャーを作成し、1 日の**平均**スループット、またはリクエストされたタイルサイズの**合計**ごとに最上位の参照元を追跡します。
- ログを**範囲フィルター**する。たとえば、[Ansible][19] タスクの実行時間のメジャーを作成し、10 秒以上かかった実行が最も多いサーバーのリストを確認します。
- その値に対して**ログを並べ替える**。たとえば、[Python][20] マイクロサービスで実行された支払い額のメジャーを作成します。その後、最も金額の高いログから順に、すべてのログを検索できます。

#### 型{#types-qualitative-facets}

メジャーには、同等の機能のために、（長）整数またはダブル値が付属しています。

#### 単位

メジャーは、クエリ時間と表示時間の桁数を簡単に処理するための単位（`seconds` の時間または `bytes` のサイズ）をサポートします。単位は、フィールドではなく、メジャー自体のプロパティです。たとえば、ナノ秒単位の `duration` メジャーを考えてみます。`duration:1000` が 1000 ミリ秒を表す `service:A` からのログと、`duration:500` が 500 マイクロ秒を表す `service:B` からの他のログがあるとします。

1. [算術演算プロセッサー][21]で流入するすべてのログの期間をナノ秒にスケーリングします。`service:A` のログには `*1000000` 乗数を使用し、`service:B` のログには `*1000` 乗数を使用します。
2. `duration:>20ms`（[検索構文][1]を参照）を使用して、両方のサービスから一度に一貫してログにクエリを実行し、最大 `1 min` の集計結果を確認します。

## ファセットパネル

検索バーを使うと、包括的かつインタラクティブにデータを分類することができます。ただし、ほとんどの場合は、ファセットパネルを使った方がよりわかりやすくデータに移動できます。ファセットを開くと、現在のクエリのスコープのコンテンツのサマリーが表示されます

**ファセット（定性的）**には、一意の値の上位リストと、それぞれに一致するログの数が用意されています。

{{< img src="logs/explorer/facet/dimension_facet.png" alt="ディメンションファセット" style="width:30%;">}}

いずれかの値をクリックして、検索クエリのスコープを設定します。値をクリックすると、この一意の値とすべての値の検索が切り替わります。チェックボックスをクリックすると、この特定の値がすべての値のリストに追加または削除されます。また、その内容を検索することもできます。

{{< img src="logs/explorer/facet/dimension_facet_wildcard.png" alt="ファセットのオートコンプリート" style="width:30%;">}}

**メジャー**には、最小値と最大値を示すスライダーが付いています。スライダーを使用するか、数値を入力して、検索クエリを別の範囲に絞り込みます。

{{< img src="logs/explorer/facet/measure_facet.png" alt="メジャーファセット" style="width:30%;">}}

### ファセットを非表示にする

ログを使用するすべてのチームのあらゆるユースケースに対処する必要があるため、組織が持つファセットは膨大になるものです。しかし、特定のトラブルシューティングのコンテキストで必要となるのは、こうしたファセットの一部のみであることが大半でしょう。トラブルシューティングセッションで最も関連性の高いファセットのみを保持するために、必要のないファセットを定期的に非表示にします。

{{< img src="logs/explorer/facet/hide_facet.png" alt="ファセットを非表示にする" style="width:30%;">}}

必要に応じて、ファセットは非表示にしてもファセット検索に表示されます（[ファセットのフィルター](#filter-facets)セクションを参照）。そこから非表示のファセットを再表示します。

{{< img src="logs/explorer/facet/unhide_facet.png" style="width:50%;" alt="ファセットを再表示" style="width:30%;">}}

非表示のファセットは、検索バーのオートコンプリートと、ログエクスプローラーの分析のドロップダウン（メジャー、グループ化など）からも非表示になります。ただし、ファセットは非表示にしても検索クエリでは有効です（たとえば、ログエクスプローラーのリンクをコピーして貼り付けた場合）。

非表示のファセットはログエクスプローラー以外には影響を与えません（例: Live Tail、モニター、ダッシュボードのウィジェットの定義）。

#### 非表示のファセットとチームメイト

ファセットの非表示は、自身のトラブルシューティングコンテキストに固有の操作で、[保存ビュー][22]を更新しない限り、チームメイトのビューには影響を与えません。非表示のファセットは、保存ビューに保存されたコンテキストの一部です。

### ファセットをグループ化

ファセットは、ファセットリスト内のナビゲーションを容易にするために、意味のあるテーマにグループ化されます。ファセットグループの割り当てや再割り当て（[ファセットの管理](#manage-facets)方法を参照）は、ファセットリストの表示に関する問題に過ぎず、検索や分析機能には影響しません。

{{< img src="logs/explorer/facet/group_facets.png" alt="ファセットをグループ化" style="width:30%;">}}

### ファセットをフィルター

ファセットの検索ボックスを使用して、ファセットリスト全体を絞り込み、操作する必要があるものにすばやく移動することができます。ファセット検索では、ファセット表示名とファセットフィールド名の両方を使用して、結果の範囲を絞り込みます。

{{< img src="logs/explorer/facet/search_facet.png" alt="ファセットを検索" style="width:30%;">}}

### エイリアスが設定されたファセット

一部のファセットにはエイリアスが設定されている場合があります（[ファセットのエイリアス設定](#alias-facets)セクションを参照）。ファセットはエイリアスが設定されても分類できますが、組織では非推奨と見なされます。

{{< img src="logs/explorer/facet/aliased_facet.png" alt="エイリアス設定されたファセット" style="width:30%;">}}

トラブルシューティングを行う場合、_エイリアス設定された_ファセットではなく_標準_ファセットで他のチームのコンテンツを（自分のチームのコンテンツと共に）見つけることが多いでしょう。これにより、さまざまな発信元からのコンテンツの関連付けがより簡単になります。

ファセットリストにエイリアス設定されたファセットが表示される場合は、**switch to alias** メニュー項目をクリックして、_標準_ファセットを使用することを検討してください。この操作により、エイリアス設定されたファセットが非表示になり、標準ファセットが再表示されます。この時に保存ビューを更新する場合は、保存ビューを保存して、この保存ビューを使用するすべてのユーザーにエイリアス設定が適用されるようにすることを検討してください。

{{< img src="logs/explorer/facet/switch_facet.png" alt="ファセットの切り替え" style="width:30%;">}}

古いコンテンツに対するトラブルシューティングを行う場合は、ファセットの非標準の_エイリアス設定された_バージョンを保持することをお勧めします（このファセットのエイリアス設定が組織によってセットアップされる前）。

## ファセットを管理

### すぐに使えるファセット

`Host`、`Service`、`URL Path`、`Duration` などの最も一般的なファセットは、ログがログインデックスに流れたらすぐに使用してトラブルシューティングを開始できます。

[予約済み属性][23]およびほとんどの[標準属性][24]のファセットがデフォルトで使用可能です。

### インデックスファセット

インデックスファセットは、組織に[複数のインデックス][25]がある場合や、アクティブな[履歴ビュー][26]がある場合にのみ表示される特定のファセットです。クエリのスコープをインデックスのサブセットに絞り込む場合は、このファセットを使用します。

{{< img src="logs/explorer/facet/index_facet_.png" alt="ファセットを作成" style="width:30%;">}}

### ファセットを作成

新しいファセットを作成するのではなく、常に既存のファセットを使用することを習慣にすると良いでしょう（[ファセットのエイリアス設定](#alias-facets)セクションを参照）。類似した性質の情報に一意のファセットを使用すると、チーム間のコラボレーションが促されます。

**注**: ファセットが作成されると、そのコンテンツは**どちらか**のインデックスに流れる**すべての新しいログに対して**入力されます。

#### ログのサイドパネルから

ファセットを作成する最も簡単な方法は、ログのサイドパネルから追加することです。フィールド名や基底のデータのタイプなど、ファセットの詳細のほとんどは事前に入力されており、再確認するだけで済みます。[ログエクスプローラー][1]で、ファセットを作成するフィールドを持つ関心のあるログに移動します。このログのサイドパネルを開き、対応するフィールド（タグまたは属性内）をクリックして、そこからファセットを作成します。

- フィールドに文字列値がある場合、ファセットの作成のみが可能です。
- フィールドに数値がある場合、ファセットとメジャーの両方を作成できます。

{{< img src="logs/explorer/facet/create_facet_from_attribute.png" alt="属性からファセットを作成" style="width:30%;">}}

#### ファセットリストから

一致するログを見つけることができない場合は、_ファセットの追加_ボタンを使用して、ファセットパネルから直接新しいファセットを作成します。

このファセットの基底のフィールド（キー）名を定義します。

- タグにはタグキー名を使用します。
- プレフィックス `@` がある、属性の属性パスを使用します。

現在のビューのログの内容に基づいてオートコンプリート機能が働くためは、適切なフィールド名を定義しやすくなっています。ただしここでは、特にインデックスに一致するログがまだ流れていない場合は、実際はどんな値でも使用できます。

{{< img src="logs/explorer/facet/create_facet_from_scratch.png" alt="一からファセットを作成する" style="width:30%;">}}

### ファセットのエイリアス設定

一意のファセットで類似するコンテンツを収集すると、チーム間の分析が可能になり、チーム間のトラブルシューティングが容易になります。[命名規則][24]を参照してください。

オプションとしてエイリアス設定を使用すると、一貫性のない命名規則に依存するチームをスムーズに再編成できます。エイリアス設定を使用すると、組織に出現する標準ファセットを使用してそれらすべてを使用できます。

#### ファセットのファセットへのエイリアス設定

これは、組織内の複数のチームが類似するコンテンツに対して複数のファセットを既に作成している場合に最適なオプションです。

_エイリアス設定された_ファセットを_標準_ファセットにエイリアス設定する場合

- ユーザーは、トラブルシューティングにエイリアス設定されたファセットと標準ファセットのどちらかを使用できます。多様で、場合によっては異種のソースから流れるコンテンツの相関が容易になる標準ファセットの方が好ましいかもしれません。
- ユーザーは、エイリアス設定されたファセットの代わりに標準ファセットを使用するように促されます。

ファセットを標準ファセットにエイリアス設定するには、ファセットメニューの `Alias to...` アクション項目を選択します。組織に存在するすべての[標準][27]ファセットから宛先ファセットを選択します。

{{< img src="logs/explorer/facet/alias_modal.png" alt="エイリアスモーダル" style="width:30%;">}}

#### 属性のファセットへのエイリアス設定

これは、新しいソースから流れるログをオンボードする場合に最適なオプションです。このログの一部のフィールドにファセットを作成するのではなく、標準ファセットにエイリアス設定してこのファセットを非推奨にした直後に、フィールドを既存のファセットに直接エイリアス設定します。

{{< img src="logs/explorer/facet/alias_facet_from_attribute.png" alt="属性からファセットをエイリアス設定する" style="width:30%;">}}

## その他の参考資料

{{< partial name="whats-next/whats-next.html" >}}

[1]: /ja/logs/search_syntax/
[2]: /ja/logs/explorer/patterns/
[3]: /ja/logs/explorer/analytics/
[4]: /ja/monitors/monitor_types/log/
[5]: /ja/dashboards/widgets/
[6]: /ja/notebooks/
[7]: /ja/logs/processing/processors/
[8]: /ja/logs/live_tail/
[9]: /ja/logs/archives/
[10]: /ja/logs/logs_to_metrics/
[11]: /ja/logs/processing/pipelines/
[12]: /ja/logs/indexes/#indexes-filters
[13]: /ja/logs/indexes/#exclusion-filters
[14]: /ja/getting_started/tagging/assigning_tags/
[15]: /ja/integrations/nginx/
[16]: /ja/logs/processing/processors/?tab=ui#geoip-parser
[17]: /ja/integrations/kong/
[18]: /ja/integrations/varnish/
[19]: /ja/integrations/ansible/
[20]: /ja/integrations/python/
[21]: /ja/logs/processing/processors/?tab=ui#arithmetic-processor
[22]: /ja/logs/explorer/saved_views/
[23]: /ja/logs/processing/#reserved-attributes
[24]: /ja/logs/processing/attributes_naming_convention/
[25]: /ja/logs/indexes/#indexes
[26]: /ja/logs/archives/rehydrating/
[27]: /ja/logs/processing/attributes_naming_convention/#standard-attribute-list
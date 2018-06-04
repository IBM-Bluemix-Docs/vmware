---
copyright:
  years: 1994, 2017
lastupdated: "2017-12-18"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# QuantaStor のアーキテクチャー・ガイド

VMware ESXi 5 環境用の OSNexus QuantaStor 共有ストレージ・ソリューションを注文して構成できます。以下の情報を『[拡張シングル・サイト VMware リファレンス・アーキテクチャー (Advanced Single-Site VMware Reference Architecture)](/docs/infrastructure/virtualization/advanced-single-site-vmware-reference-architecturesoftlayer.html)』クックブックとともに使用して、VMware 環境で以下のストレージ・オプションのいずれかをセットアップします。

## ステップ 1: QuantaStor 共有ストレージの注文

ストレージを注文する前に、お客様の容量と入出力の要件を満たす仕様を判別する必要があります。この仕様には、サーバーの RAM、ディスク・ドライブのタイプと数、キャッシュ用の SSD、RAID 構成、ネットワーク構成などが含まれます。ただし、これらに限定されるわけではありません。ご使用の環境での適切な QuantaStor 構成の選択について詳しくは、[QuantaStor Solution Design Guide for Virtualization ![外部リンク・アイコン](../../icons/launch-glyph.svg "外部リンク・アイコン")](http://wiki.osnexus.com/index.php?title=Solution_Design_Guide#Server_Virtualization){: new_window} を参照してください。

環境の例として、仮想マシン (VM) で必要とされる十分な容量と入出力を提供できる小規模構成を使用します。注文する前に、ご使用の VM の容量と入出力の要件を必ず理解してください。QuantaStor サーバーは拡張可能ですが、構成およびデプロイメントで遅延が生じないように、最初にハードウェアを見積もることが重要になります。

### QuantaStor の注文

以下のステップを使用して、QuantaStor サーバーを注文します。

1. [{{site.data.keyword.slportal_full}} ![外部リンク・アイコン](../../icons/launch-glyph.svg "外部リンク・アイコン")](https://control.softlayer.com/){: new_window} にログインし、**「アカウント」>「注文」**をクリックします。
2. ポップアップ画面で {{site.data.keyword.baremetal_short}}、月単位を選択します。
3. 「サーバー・リスト」ページで、ご使用の環境で必要なディスクの数を保管できる適切なサーバーを選択します。[環境の例では、12 個のドライブ・ベイを搭載した 12 コアのシステム (つまり、デュアル 6 コア) が選択されています]。
4. 以下の構成オプションを入力します。
  * **データ・センター:** 以前に作成した VLAN および ESXi ホストの場所
  * **サーバー:** デュアル・プロセッサー 6 コア Xeon
  * **RAM:** 64 GB
  * **オペレーティング・システム:** OSNexus QuantaStor 3.x (4 TB)
  * **ハード・ディスク:**
    * ディスク 1 & 2: RAID 1 の 500 GB SATA
    * パーティション・テンプレート: Linux 基本
    * ディスク 3-10: RAID-10 の 600 GB SAS 15K
  * **パブリック帯域幅:** プライベート・ネットワークのみ
  * **アップリンク・ポート速度:** 10 Gbps 冗長プライベート・ネットワーク・アップリンク
5. **「注文を続行 (Continue Your Order)」**をクリックします。

**注:** 2 つの異なるサブネットを使用してストレージ・アレイに対するトラフィックをロード・バランシングできるように、結合されていない 2 つのネットワーク・インターフェースを使用して、ストレージ・サーバーは構成されます。

### 構成の終了

これで、ショッピング・カートに QuantaStor アプライアンスが入りました。次に、プライベート VLAN、ホスト名、およびドメインをデバイスに割り当てて、デバイスを正しくプロビジョンできるようにする必要があります。

1. 次の VLAN を割り当て、デバイスのホスト名を作成します: QuantaStor ホスト – バックエンド VLAN: (この環境では 1102)
2. 支払い方法を選択し、ご利用条件に同意し、「注文を確定する」をクリックします。

**注** サーバーがプロビジョンされ、VPN または仮想サーバーを介してアクセス可能になるまで、ステップ 3 に進まないでください。

## ステップ 2: 管理ホストおよび容量ホストのための iSCSI ソフトウェア・アダプターの有効化

iSCSI ボリュームをマウントする前に、管理および容量の各ホストで iSCSI ソフトウェア・アダプターを有効にし、その iSCSI 修飾名 (IQN) を収集しておく必要があります。以下のステップを使用して、iSCSI ソフトウェア・アダプターを有効にします。

1. ESXi の管理ホストまたは容量ホストにアクセスし、「管理 (Manage)」、「ストレージ (Storage)」、「ストレージ・アダプター (Storage Adapters)」を選択します。
2. 緑の正符号 (+) をクリックし、iSCSI ソフトウェア・アダプターを追加します。
3. iSCSI ソフトウェア・アダプターに対応する vmhba をクリックし、有効になった後に、「アダプターの詳細 (Adapter Details)」セクションの iSCSI 名 (図 1) を記録します。
4. ESXi の各管理ホストおよび容量ホストで、iSCSI ソフトウェア・アダプターごとにステップ 1 から 3 を実行します。

## ステップ 3: QuantaStor の構成

サーバーがプロビジョンされた後に、共有ストレージ・デバイスをセットアップできます。具体的には、ネットワーキングをセットアップし、iSCSI ボリュームを構成し、ボリュームをホストに割り当てます。 

例えば、ボリューム vmk3 には、ストレージ VLAN 上のポータブル・プライベート・サブネット A に接続する vmnic0 があります。ボリューム vmk4 には、ストレージ VLAN 上のポータブル・プライベート・サブネット B に接続する vmnic2 があります。QuantaStor サーバーには、ストレージ VLAN に接続する 2 つのプライベート・ネットワーク・アダプター eth4 および eth6 もあります。

### ネットワーキングの構成

1. Web ブラウザーを開き、[{{site.data.keyword.slportal_short}} ![外部リンク・アイコン](../../icons/launch-glyph.svg "外部リンク・アイコン")](https://control.softlayer.com/){: new_window} の**「デバイス」**ページにリストされている QuantaStor IP アドレスにアクセスします。
2. **管理者ユーザー名**と**パスワード** (**「デバイスの詳細」**ページの「パスワード」セクションに記載) を入力します。先へ進む前に、QuantaStor サーバーで使用されているプライベート・ネットワークデバイス (eth4 など) をメモします。
3. **「ストレージ・システム (Storage System)」>「ネットワーク・ポート (Network Ports)」**に移動します。
4. ネットワーク・アダプターのリストで、アクティブなプライベート・アダプター (例: eth4) を選択します。右クリックし、ポップアップ・メニューで**「ネットワーク・ポートの変更 (Modify Network Port)」**を選択します。
5. **「iSCSI 有効 (iSCSI Enabled)」**チェック・ボックスをクリアして、このアダプターへの iSCSI 接続を無効にし、**「OK」**をクリックします。
6. アクティブでないが、プライベート・ネットワークに割り当てられているプライベートアダプター (例えば、eth6) を選択します。
7. eth6 アダプターを右クリックし、ポップアップ・メニューで**「ネットワーク・ポートの変更 (Modify Network Port)」**を選択します。
8. **「ネットワーク・ポートの変更 (Modify Network Port)」**画面で**「構成タイプ (Config Type)」**として**「静的 (Static)」**を選択します。
9. アダプターの 1 次プライベート IP アドレス、サブネット、ゲートウェイを入力します。『[拡張シングル・サイト VMware リファレンス・アーキテクチャー (Advanced Single-Site VMware Reference Architecture)](/docs/infrastructure/virtualization/advanced-single-site-vmware-reference-architecturesoftlayer.html)』クックブックの VLAN ワークシートを使用する場合、「ストレージ (Storage)」行のアドレスを使用します。
10. **「iSCSI 有効 (iSCSI Enabled)」**チェック・ボックスをクリアして、このアダプターへの iSCSI 接続を無効にします。
11. プライベート・アダプターを右クリックし、**「ネットワーク・ポートの有効化 (Enable Network Port)」**を選択し、アダプターをオンラインにします (IP アドレスを指定してアダプターを構成した後)。
12. **「OK」**をクリックします。IP アドレスが検証されると、ポートが有効になります。

### サポート・チケットのオープン

1 次プライベート IP アドレスを使用して 2 つ目のアダプターを構成した後に、サポート・チケットをオープンする必要があります。チケットをオープンすると、使用した IP アドレスが、別のマシンを VLAN でプロビジョンした場合に確実に使用されないようにすることができます。

1. **「サポート」**>**「チケットの追加」**をクリックし、以下の情報を入力します。
  * 件名: プライベート・ネットワークの質問
  * タイトル: プライベート IP アドレスの予約および割り当て
  * デバイスの関連付け:「QuantaStor サーバーの選択」
  * 詳細: VLAN で予約し、割り当ててください。この IP アドレスは、QuantaStor サーバー上の 2 つ目のアダプターで使用されます。

### QuantaStor の構成

アレイで両方のアダプターからの接続を受け入れることができるようになったので、次は、ストレージ・パス A サブネットとストレージ・パス B サブネット上にあるアダプターの仮想インターフェースを割り当てる必要があります。

  1. 最初のプライベート・アダプター・インターフェース (eth4) を右クリックし、ポップアップ・メニューで**「仮想インターフェースの作成 (Create Virtual Interface)」**を選択します。
  2. 結果として表示されたポップアップ画面でアダプターのポータブル・プライベート IP アドレスおよびサブネットを入力し、**「iSCSI 有効 (iSCSI Enabled)」**にチェック・マークが付けられていることを確認します。
  3. VLAN ワークシートを使用する場合は、「ストレージ・パス A (Storage Path A)」行のアドレスを使用します。
  4. 「ポータブル IP アドレス (Portable IP address)」ページの「メモ (Notes)」セクションで使用されている IP アドレスを記録します。
  5. 最初のプライベート・アダプター・インターフェース (eth4) を選択して仮想インターフェースをバインドし、**「OK」**をクリックします。
  6. もう 1 つのプライベート・アダプター・インターフェース (eth6) を右クリックし、ポップアップ・メニューで**「仮想インターフェースの作成 (Create Virtual Interface)」**を選択します。
  7. 結果として表示されたポップアップ・ウィンドウで、アダプターのポータブル・プライベート IP アドレスおよびサブネットを入力し、**「iSCSI 有効 (iSCSI enabled)」**にチェック・マークが付けられていることを確認します。
  8. VLAN ワークシートを使用する場合は、「ストレージ・パス B (Storage Path B)」行のアドレスを使用します。
  9. 「ポータブル IP アドレス (Portable IP address)」ページの「メモ (Notes)」セクションで使用されているこの IP アドレスを記録します。
  10. もう 1 つのプライベート・アダプター・インターフェース (eth6) を選択して仮想インターフェースをバインドし、**「OK」**をクリックします。

IP アドレスおよび仮想インターフェースを使用して QuantaStor サーバーを構成した後に、ルーティングを構成して、(異なるサブネット上に 2 つの NIC があるため) 出力トラフィックが正しいインターフェースを確実に使用するように構成する必要があります。

1. root 資格情報を使用して、QuantaStor サーバーに SSH で接続し、以下の行を /etc/network/interfaces に追加します。eth4 および eth6 が、プライベート・ネットワーク上の NIC であると想定しています。
  * post-up ip route add 10.0.0.0/8 via dev eth4
  * post-up ip route add 10.0.0.0/8 via dev eth6 table eth6
  * post-up ip rule add from table eth6

### ストレージ・プールの作成

次に、ボリュームや共有を作成するには、その前に、ボリュームの割り振りに使用するストレージ・プールを作成する必要があります。

1. **「ストレージ・プール (Storage Pools)」**を右クリックし、**「ストレージ・プールの作成 (Create Storage Pool)」**を選択します。
2. **「ストレージ・プールの作成 (Create Storage Pool)」**画面で以下の情報を入力します。
  * 名前 (Name): ストレージ・プールの適切な名前を入力します。例: StoragePool-01
  * プール・タイプ (Pool Type): デフォルト (Default) (zfs)
  * 入出力プロファイル (I/O Profile): 仮想化 (Virtualization)
  * ストレージ・プールで使用するディスク (Disks to Utilize for the Storage Pool): sdb
3. **「OK」**をクリックします。

sdb がプールへの追加に使用できない場合、ブート可能とタグ付けられているパーティションが存在していることが原因です。Quantastor コンソールで gdisk を使用して、そのパーティションおよびすべての /etc/fstab 項目を削除する必要があります。次に、ESXi が取り込むボリュームを作成する必要があります。

### iSCSI ストレージ・ボリュームの作成

2 つのストレージ・ボリュームを作成する必要があります。 1 つのボリュームは、管理クラスター上の管理 VM 用に使用され、もう 1 つは、容量クラスター上の VM 用に使用されます。以下のステップを実行して、iSCSI ストレージ・ボリュームを作成します。

1. **「ストレージ・ボリューム (Storage Volumes)」**を右クリックし、ポップアップ・メニューで**「ストレージ・ボリュームの作成 (Create Storage Volume)」**を選択します。
2. ストレージ・ボリュームの情報を入力します。**注:** ご使用の構成は、ワークロードおよびストレージ容量に応じて異なる可能性がありますが、例では、表 1 と表 2 に各ストレージ・ボリュームの値を示します。

|フィールド|値|
|---|---|
|名前 (Name)|Mgmt-LUN0|
|ストレージ・プール (Storage Pool)|[前のステップで構成したストレージ・プール名]|
|サイズ (Size)|500GB|
|予約済み % (% Reserved)|50|
|圧縮 (Compression)|有効 (Enabled)|
{: caption="表 1. iSCSI 管理ボリューム" caption-side="top"}

|フィールド|値|
|---|---|
|名前 (Name)|Capacity-LUN0|
|ストレージ・プール (Storage Pool)|[前のステップで構成したストレージ・プール名]|
|サイズ (Size)|3TB|
|予約済み % (% Reserved)|50|
|圧縮 (Compression)|有効 (Enabled)|
{: caption="表 2. iSCSI 容量ボリューム" caption-side="top"}

### ホストに対するボリュームへのアクセス権限の割り当て

ボリュームのセットアップ後に、各 ESXi ホストの IQN を介した ESXi ホストからのアクセスを許可するように QuantaStor を構成する必要があります。

1. QuantaStor 管理ページにナビゲートし、**「ホスト (Hosts)」**メニューを右クリックし、**「ホストの追加 (Add Host)」**を選択します。
2. **「ホストの追加 (Add Host)」**画面で以下の情報を入力します。
  * ホスト名 (Host Name): 適切なホスト名を入力します。ホスト名は、完全修飾ドメイン名 (FQDN) ではありませんが、そのホストを説明するものでなければなりません。例: MyESXiHostName
  * オペレーティング・システム・タイプ (Operating System Type): VMware
  * イニシエーター (Initator): 「iSCSI 修飾名 (IQN) (iSCSI Qualified Name (IQN))」ラジオ・ボタン
  * iSCSI 修飾名 (iSCSI Qualified Name): 各ホストの IQN。
3. **「OK」**をクリックします。

ESXi 環境内の管理ホストおよび容量ホストごとに、上記ステップに従います。

管理クラスターおよび容量クラスター内の各ホストを追加した後に、以下のステップに従います。

1. **「ホスト・グループ (Host Groups)」**メニューを右クリックし、**「ホスト・グループの作成... (Create Host Group…)」**を選択します。
2. **「名前 (Name)」**フィールドに「ManagementCluster」と入力し、管理クラスター内のすべてのホストを選択します。
3. **「OK」**をクリックします。特定のボリュームに割り当てることができるホスト・グループが作成されます。

容量クラスターについて、このプロセスを繰り返します。

1. **「ストレージ・ボリューム (Storage Volumes)」**メニューをクリックします。
2. **「Mgmt-Lun0」**ボリュームを右クリックし、**「ホストに対するアクセスの割り当て/割り当て解除 (Assign/Unassign Host Access)」**を選択します。
3. **「Mgmt-Lun0」**がドロップダウン・メニューのオプションであることを確認し、前のステップで作成したホスト・グループを選択します。このオプションにより、管理クラスター内の各 ESXi ホストが Mgmt-Lun0 ボリュームにアクセスできるようになります。**Capacity-LUN0** に対して同じ処理を実行します。

## ステップ 4: 管理/容量クラスターでのボリュームのマウント

vSphere Web Client にログインし、**「vCenter インベントリー・リスト (vCenter Inventory Lists)」」**の下の**「ホスト (Hosts)」**に移動します。

以下のステップを使用して、ESXi ホストでボリュームをマウントします。

1. ホストを選択し、**「管理 (Manage)」、「ストレージ (Storage)」**>**「ストレージ・アダプター (Storage Adapters)」**をクリックします。
2. **「ターゲット (Targets)」**>**「動的ディスカバリー (Dynamic Discovery)」**>**「追加... (Add…)」**を選択します。
3. **「送信ターゲット・サーバーの追加 (Add Send Target Server)」**画面で、ストレージ・パス A 上の QuantaStor ストレージ・デバイスに割り当てられている IP アドレスを入力します。
4. ポートを **3260** のままにし、**「OK」**をクリックします。
5. **「動的ディスカバリー (Dynamic Discovery)」**>**「追加... (Add…)」**をクリックします。ストレージ・パス B について、ステップ 4 と 5 を繰り返します。

ESXI ホストが、Mgmt-Lun0 ボリュームをディスカバーするために iSCSI ソフトウェア・アダプターを再スキャンする準備ができました。

1. **「ストレージ・アダプター (Storage Adapters)」**画面で**「ストレージ・アダプターの再スキャン (Rescan Storage Adapters)」**アイコン (図 9) を選択します。
2. 結果として表示されたポップアップ画面でデフォルト・オプションのままにして、**「OK」**をクリックします。
3. ボリュームのディスカバー後に**「アクション (Actions)」、「新規データ・ストア (New Datastore)」**をクリックし、**「VMFS-5」**としてフォーマットします。
4. フォーマットが完了したことを確認し、**「ストレージ・デバイス (Storage Devices)」、「デバイスの詳細 (Device Details)」、「マルチパスの編集... ( Edit Multipathing…)」**をクリックします。
5. **「パス選択ポリシー (Path Selection Policy)」**で**「ラウンドロビン (Round Robin)」**を選択し、**「OK」**をクリックします。

これで、Mgmt-Lun0 ボリュームがシングル・ホストに接続されたので、次は、クラスター内の他の管理ホストに戻って、ボリュームを追加するプロセスを繰り返す必要があります。ただし、ボリュームは既に VMFS-5 で既にフォーマットされており、ディスカバリー後にそのように表示されるため、ボリュームをフォーマットする必要はありません。

## ステップ 5: vSphere ESXi ホストでの遅延 ACK の無効化

ストレージ LUN が管理ホストおよび容量ホストに接続された後に、遅延 ACK を無効にする必要があります。

1. vSphere 環境にアクセスし、**「ストレージ・アダプター (Storage Adapters)」**>**「iSCSI ソフトウェア・アダプター (iSCSI Software Adapter)」**>**「詳細オプション (Advanced Options)」**を選択します。
2. **「編集 (Edit)」**をクリックし、「詳細オプション (Advanced Options)」画面の末尾までスクロールします。
3. **「遅延 Ack (DelayedAck)」**チェック・ボックスをクリアし、**「OK」**をクリックします。

VMware に関する遅延 ACK について詳しくは、[VMware の知識ベース](https://kb.vmware.com/s/article/1002598)を参照してください。

これで、『[拡張シングル・サイト VMware リファレンス・アーキテクチャー (Advanced Single-Site VMware Reference Architecture)](/docs/infrastructure/virtualization/advanced-single-site-vmware-reference-architecturesoftlayer.html)』クックブックに戻り、環境のセットアップを実行できます。
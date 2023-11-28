# aws-vpcflowlog-cloudwatch 概要
AWS VPCflowlog/CloudWatch/AWS SNSを使用してネットワーク監視システムを構築します。

## Architecture Diagram
![vpcflowlog drawio](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/4e935739-dd0c-459a-bf6a-ac87c19e6848)

## 前提条件
- AWSにて適切なユーザーアカウントがあること
- 適切なVPC, サブネットリソースが作成済み
- VPCにInternet Gatewayをアタッチ済み
- VPCのルートテーブルを作成済み
- 適切なセキュリティグループロールを付与済み

## VPCフローログを作成する

**ログ グループ**をクリックし、 **ログ グループの作成**をクリック。詳細を入力します。
  * ログ グループ名: Log-Groupe
  * 保持設定: [失効しない(無期限)]を選択
  * 他のオプションはデフォルトのままにして、 「作成」ボタンをクリック
     ![スクリーンショット 2023-11-22 14 30 33](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/d13280cf-a5e2-4763-b94c-f60ff54a17f4)



次にVPCフローログを構築します。  
* 上部の**[サービス]**メニューに移動し、 **VPC**を選択します。
* 作成済みの**VPC**を選択し、**フロー ログ**タブをクリック
![スクリーンショット 2023-11-22 14 32 12](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/1bf60179-683e-4fe9-b9a6-37b31ab34273)


* 値を入力してVPCフローログを作成
  * 名前:　Flowlog-Cloudwatch
  * フィルター:　すべて
  * 最大集計間隔:　1分
  * 宛先:　CloudWatch Logs に送信
* 宛先ロググループ: 先ほど作成したLog-Groupを選択します
* [フロー ログの作成]をクリック

VPCフローログが正常に作成されました。  
![fl4](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/2e8f538a-b7e2-4a38-920b-600cf3eed261)

## 検証用のEC2インスタンスを作成する

* Amazon マシンイメージ (AMI) : **Amazon Linux 2 AMI**
* インスタンスタイプ: **t2.micro**
* 前もって作成済みのVPCを追加
* パブリックIPの自動割り当て: **[有効にする]**
* ファイアウォール (セキュリティ グループ) : [既存のセキュリティグループを選択]
* 共通セキュリティグループ: 前もって作成済みのセキュリティグループを選択

* それ以外はすべてデフォルトのまま「インスタンスの起動」ボタンをクリック
* 1～2分後、インスタンスの状態が**running**になります

## ロググループ内のログを確認する

EC2インスタンスに接続し、Google サーバーに``$ ping``を実行します。実行後、CloudWatchロググループにログが生成されています。  

* EC2InstanceのインスタンスIDをクリックし、**接続**をクリック
* **EC2 Instance Connect**タブで、**Connect**をクリック
![スクリーンショット 2023-11-22 14 39 09](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/d906364d-bbbc-4012-8f0f-b2dac965bf3f)

* ICMPプロトコル経由でトラフィックを送信してみます(google.comにpingを送信します)
  * ターミナルウィンドウに``$ ping google.com``を実行
  * 複数のパケットを受信したら、**Ctrl+C**で停止(Mac OSの場合)
  ここで``$ ping``応答に含まれる**google.comのIPアドレス**をコピーしておきます(スクショの例から142.251.111.102)
![スクリーンショット 2023-11-22 14 39 59](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/52cbf6d7-79d2-4489-83a5-8740133d619c)

* CloudWatchロググループに移動
* [ロググループ]をクリックし、先ほど作成した[Log-Group]を選択
* [ログストリーム]の下に、EC2インスタンスのeniログがあるので、ログを選択
![スクリーンショット 2023-11-22 14 40 55](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/2ab3f15f-135f-48e7-b425-cfb45bee8e5d)

* [ログイベント] で、**GoogleのIPアドレス**(前の手順でコピーしたもの)を入力してログを検索します。
* VPCフローログに一致するトラフィックレコードが表示されています。(タイムスタンプに**ACCEPT**として表示されていることにも注意
![スクリーンショット 2023-11-22 14 43 15](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/137cdcf8-7a4d-4595-a256-30fb4f46f3d2)


## トラフィックを監視するためにCloudWatchメトリクスフィルターとCloudWatchアラームを構築する
最初に**ネットワークACLでICMPトラフィックを拒否**設定します。その後、EC2インスタンスからのトラフィックを監視するメトリクスフィルターを設定。  
フィルターで**拒否されたpingが検出されるたびにSNS通知経由でメールを送信する**CloudWatchアラームを作成。


* [ネットワークとコンテンツ配信]で[VPC]を選択
* [ネットワーク ACL]を選択
* **Subnet**に関連付けられている**ネットワークACL**を選択
* [アウトバウンドルールの編集]ボタンをクリック
![スクリーンショット 2023-11-22 14 44 28](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/328ab782-75ab-4dd9-bee5-22e3f9d0de13)

* [新しいルールの追加]をクリックし、ルールを追加します
* ルール番号: 50 (**ルールは低い番号から優先されていきます**)
* タイプ: **ICMP - IPv4**
* 許可/拒否: 拒否 
![スクリーンショット 2023-11-22 14 46 48](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/47269e7b-f2e7-4e65-9398-bad16501b19f)

* 次に[CloudWatch]に移動し[ロググループ]をクリック
* 先ほど作成した[Log-Group]を選択
* [メトリクスフィルター]タブで、[メトリクスフィルターの作成]をクリック
![スクリーンショット 2023-11-22 14 48 00](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/9d44beee-3c3f-46da-a6cc-c534a9a81423)  


* フィルターパターンを作成します
  * フィルターパターン: [version, account, eni, source, destination, srcport, destport, protocol="1", packets, bytes, starttime, endtime, action="REJECT", flowlogstatus]
 
| **設定項目** | **内容** |
|---|---|
| 名前 | 任意のフローログ名。 |
| フィルター | アクセスの許可(ACCEPT)、拒否(REJECT)、両方から選択。ログには"action"として表示。 |
| 最大集計期間 | 10分か1分で選べる。1分の方がログを出力するまでの時間が短いため、すぐに確認する必要がある場合に使用。 |
| 送信先 | CloudWatchLogs。 |
| 送信先ロググループ | 前述のロググループ。 |
| IAMロール | 前述のIAMロール。 |
| ログレコードの形式 | デフォルトかカスタムを選択。カスタムは選択した順番がログに反映される。形式のプレビューで順番を確認可能。 |

詳細なメトリクスフィルター、フィルターパターン構文の参照はこちら  
AWS公式 - メトリックスフィルター、サブスクリプションフィルター、フィルター ログイベントのフィルターパターン構文
 https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/logs/FilterAndPatternSyntax.html

![スクリーンショット 2023-11-22 14 53 39](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/6ba54ebc-7563-46fc-8abc-5f1f08fc0261)
* メトリクスの詳細:
  * メトリクス名前空間: Ping-Deny-Metrics
  * メトリクス名: Ping-Deny
  * メトリクス値: 1
  * 他のオプションはデフォルトのままにして、 [次へ]

先ほど作成したメトリクスフィルター[Ping-Deny-Metrics]を選択し、[アラームを作成]をクリック
![スクリーンショット 2023-11-22 14 54 44](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/9753e817-cb58-410a-93bb-5666f086cc77)

* メトリクス名: Ping-Deny
* 統計: 合計
* 期間：1分
* 条件セクション:
  * しきい値の種類: 静的
  * **Ping-Deny**が次の時…: 以上/>= しきい値
  * ...よりも: 1
![スクリーンショット 2023-11-22 14 55 31](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/12071a38-d54d-4b36-bdec-ef10e3939d74)

![スクリーンショット 2023-11-22 14 55 58](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/0902db47-75aa-44f2-ac4c-90ea8566a76a)

* 次にアラームアクションの値を入力します。
![スクリーンショット 2023-11-22 14 56 56](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/d995933d-89e3-46b2-bab6-23d7bc349624)
  * アラーム状態トリガー: アラーム状態
  * SNS トピック: 新しいトピックの作成
  * 新規トピックの作成中…: CloudWatch-Alarms-Ping-Deny
  * 通知を受信するEメールエンドポイント…: (受け取りたいメールアドレスを入力)
  * **「トピックの作成」**ボタンをクリック
  他のオプションは**デフォルト**のままで、次へいきます

![スクリーンショット 2023-11-22 14 57 30](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/e0e64733-eafb-4995-a358-00de8c5c1971)  
* アラーム名: CloudWatch-Alarm-Ping-Deny

## CloudWatch アラームをテストする
ICMPリクエストを送信してアラームテストをします。  
* Googleサーバーにpingを送信し、CloudWatchアラームがメールを送信するかどうかをテストします。

* EC2InstanceのインスタンスIDをクリックし、**接続**をクリック
* **EC2 Instance Connect**タブで、**Connect**をクリック
* ``$ ping google.com``を実行して、 ICMP プロトコル**経由でトラフィックを送信してみます
  * **ネットワークACL**にはアウトバウンド**ICMP**トラフィックに対する拒否ルールがあるため、応答は受信されません
  * 受信しない事を確認したので**Ctrl+C**で実行を終了します
![スクリーンショット 2023-11-22 15 00 13](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/86cd5d99-272a-4d8b-a912-66aacc1724df)

* CloudWatchメトリクスフィルターの**REJECT**イベントが発生し、 **CloudWatchアラーム**がトリガーされ、  
先ほど作成したアラーム**CloudWatch-Alarm-Ping-Deny**を選択すると、
アラーム状態が[アラーム中]に変わっています。(数分間待つ必要があります)

* 先ほど作成したSNSトピックサブスクリプションにて、アラームの詳細が記載されたメールが届きます。
![スクリーンショット 2023-11-22 15 02 19](https://github.com/Kana-Karin/aws-vpcflowlog-cloudwatch/assets/84316229/7146318b-5a83-4ef7-bfe2-b71807f36d33)

以上によりCloudWatchアラームが正常に動作されていることが確認できました。  
お疲れさまでした🙇‍♀️

## リソースの削除
- EC2インスタンスの削除(アクション→終了)
- CloudWatchロググループの削除
- CloudWatchメトリクスの削除
- VPCリソースの削除
- セキュリティグループ、ネットワークACL設定の削除

## 参照させていただいたサイト






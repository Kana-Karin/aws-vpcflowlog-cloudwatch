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




製作中...
-----------------






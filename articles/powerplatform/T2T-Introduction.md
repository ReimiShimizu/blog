---
title: 環境のテナント移行について
date: 2022-12-28 17:30:00
tags:
  - Power Platform
  - テナント移行
  - 環境
---

#  環境のテナント移行について

こんにちは、Power Platform サポートチームの島です。
本記事では、Power Platform の Dataverse あり環境を他のテナントに移行するプロセスについてご案内します。
<!-- more -->

## 概要

Power Platform では、複数テナントの統合や企業組織の変更などに対応するため、環境のテナント移行をサポートしています。
テナント移行は運用影響の大きい処理であり、また __リハーサル日程も含めた形で事前に移行スケジュールを計画いただく__ ことを弊社として強く推奨しております。
そのため計画段階でのお役に立てるよう、移行全体の流れやよくいただくご質問についてご案内いたします。

なお記事の前提として、以下の点についてご留意ください。
* 本情報の内容（添付文書、リンク先などを含む）は、作成日時点でのものであり、予告なく変更される場合があります。
特にテナント移行は環境のダウンタイムを伴う運用影響の大きい処理ですので、確実に移行を完遂するため、記載内容についてはお手元でのご検証や弊社への問合せなどでもご確認ください。
* 最新情報につきましては、以下の公開情報もご参照ください。
[テナントからテナントへの移行](https://learn.microsoft.com/ja-jp/power-platform/admin/move-environment-tenant)
* テナント移行は Dataverse あり環境が対象です。Dataverse なしの環境についてはテナント移行の実施が叶いませんので、Dataverse なしの環境に含まれるキャンバス アプリや Power Automate などの移行方法につきましては、各製品の公開情報をご参照ください。

---
1. [移行の流れ](#anchor-whole-process)
2. [よくいただくご質問](#anchor-QA)

<a id='anchor-whole-process'></a>
## 1. 移行の流れ
テナント移行は以下の流れで実施いたします。

1. (お客様) 移行対象の本番環境を直近でコピーしている環境について、リハーサル移行用のお問合せを起票いただく。
2. (お客様) テナント移行の事前準備を進めていただく。
3. (お客様) ユーザー マッピングファイルをご提供いただく。
4. (弊社) ユーザー マッピングファイルの検証を実施。
5. (弊社) 日程調整の上、テナント移行を実施。
6. (お客様・弊社) 1 ～ 5 のプロセスにて、リハーサル後の移行環境の動作に問題がなかった場合には、本番環境に対して 1 ～ 5 のプロセスを実施。

### 1-1. お問合せの起票について
本番環境をコピーしたリハーサル環境をご用意いただいた上で、移行対象の環境ごとに、テナント移行を希望いただいている旨をご連絡ください。
お問合せ起票時に、以下の情報をお寄せください。
* 移行対象環境の環境 URL
* 移行先のテナント ID ※[ご確認方法](https://learn.microsoft.com/ja-jp/sharepoint/find-your-office-365-tenant-id)
* 移行先テナントで環境にセキュリティ グループを割り当てる場合、セキュリティ グループの Azure Active Directory Object ID
* 移行実施をご希望の日時

なお、移行日時をご指定いただく場合には、実施の 1 週間前までにお問合せにて日時をお知らせください。

### 1-2. テナント移行の事前準備について
以下の項目について対応を実施ください。

##### 1-2-1. 移行先テナントのライセンスについて
移行先テナントで、事前にユーザーに環境の利用権を有する Power Platform ライセンスを割り当てください。
またテナント単位のライセンスや Power Apps per app ライセンスをお使いの場合など、ユーザーに割り当てるライセンスがない場合にはお問合せにて個別にご相談ください。

##### 1-2-2. 移行先テナントの容量について
移行先テナントにて、移行対象の環境と同等以上の Power Platform テナントの空き容量を確保ください。
もしやむを得ず空き容量の確保が難しい場合には、お問合せにて個別にご相談ください。

<a id='anchor-unsupported-components'></a>
##### 1-2-3. 事前準備が必要なコンポーネントについて
テナント移行にあたって事前準備を実施いただく必要のあるコンポーネントがございます。
各コンポーネントについて、環境上での利用有無をご確認いただき、利用いただいている場合には事前準備を実施ください。
・[ご参考 : 対象コンポーネントについて](https://learn.microsoft.com/ja-jp/power-platform/admin/move-environment-tenant?tabs=image#confirm-if-any-of-the-solutions-below-are-installed-in-the-environments-to-be-migrated-as-these-may-require-additional-steps-either-from-you-or-support)

なお上記のコンポーネントの中には、キャンバス アプリや Power Automate など、__事前対応いただかなかった場合移行できず消滅する機能__ が複数存在いたします。
そのため対象機能については、移行後に正しく動作するかをリハーサルにて必ずご確認いただきますようお願い申し上げます。

##### 1-2-4. ユーザー (SystemUser) テーブルのプラグインについて
ユーザー (SystemUser) テーブルに対してプラグインを実装いただいている場合、当該プラグインが動作することでユーザー マッピング時に想定しない処理が発生する可能性がございます。
そのため、移行前にユーザーテーブルのプラグインを無効化いただきますようお願い申し上げます。

### 1-3. ユーザー マッピングファイルのご提供について
ユーザー マッピングファイルは、テナント移行において移行元テナントのユーザーと移行後テナントのユーザーを紐づける定義であり、以下のフォーマットで記述された CSV ファイルです。
「_移行元テナントのユーザーの UPN, 移行先テナントのユーザーの UPN_」

例えば AAA.com テナントから BBB.com テナントへ移行を行い、ユーザー XXX および YYY の 2 名のマッピングを行う場合、ユーザー マッピングファイルは以下のように記述されます。
_`XXX@AAA.com,XXX@BBB.com`_
_`YYY@AAA.com,YYY@BBB.com`_

そのため、以下の手順に従ってユーザー マッピングファイルを作成し、お問合せ内でご提供ください。
・[ご参考 : マッピング ファイルを作成するステップ](https://learn.microsoft.com/ja-jp/power-platform/admin/move-environment-tenant?tabs=image#steps-to-create-the-mapping-file)

なお、ユーザー マッピングファイルの内容に起因してテナント移行が失敗する事例が多くございます。
そのため、__リハーサル移行であっても極力本番移行と同一のユーザー マッピングファイル__ をご提供いただきますようお願い申し上げます。
本番移行まで期間が空くなどにより同一ファイルのご提供が難しい場合には、観点を分けての複数回のリハーサル実施も承っております。

### 1-4. ユーザー マッピングファイルの検証について
ユーザー マッピングファイルをご提供いただいた後、弊社にてファイルの内容をもとに、ユーザーの存在有無やライセンス状況の確認を行います。
問題があった場合にはお問合せ内でその旨をご連絡いたします。
問題がない場合には、調整させていただいた日時をもとに、テナント移行の実施を手配いたします。

### 1-5. テナント移行の実施について
いただいた情報をもとに、弊社データセンター部門にてテナント移行を実施いたします。
テナント移行の実施中は環境にアクセスいただけなくなりますのでご注意ください。
移行完了後、技術サポートから窓口営業時間内に移行結果をご連絡いたします。

### 1-6. 本番移行の実施について
弊社では、本番環境をテナント移行する場合、移行が適切に行えることを確認するために必ずテスト環境での事前リハーサルをお願いしております。
必ず 1 から 5 までの手順にてリハーサル結果が問題ないことを確認の上で、本番環境の移行をご依頼ください。
特に [1-2-3](#anchor-unsupported-components) の通り、移行にあたって追加対応が必要なコンポーネントにつきましては、必ずリハーサル後に動作をご確認ください。

本番移行にあたっては、リハーサルを実施した際のお問合せ番号を併記の上、改めてお問合せを起票いただけますと幸いです。

---
<a id='anchor-QA'></a>
## 2. よくいただくご質問

### 2-1. テナント移行前後で変更される環境情報はあるか
以下の項目については変更や影響がございますので、ご留意ください。
* [事前準備が必要なコンポーネント](#anchor-unsupported-components)には移行されないものがございます。
* ユーザー マッピングに伴い、UPN や氏名などのユーザー情報が新テナントの Azure Active Directory の内容に置き換わります。

以下の項目は変化いたしません。
* 環境の URL ※環境を別リージョンに移行する場合を除く
* 環境の組織 ID
* モデル駆動型アプリの内容や構成
* 関連付けやレコードの所有権情報、セキュリティ ロール情報などを含む Dataverse のすべてのデータ

### 2.-2. 移行の所要時間はどの程度か
移行の所要時間は、クラウド リソースの利用状況やネットワーク状況に依存いたします。
そのため移行時間を事前にお見積りすることは叶いませんが、通常では数時間程度の所要時間をいただいております。
詳細な移行時間につきましては、リハーサル移行の実績を目安としていただけますと幸いです。

### 2-3 夜間や休日の移行を希望するが、Microsoft 側でどのような支援体制が可能か
弊社技術サポートでは、[弊社技術サポート営業時間](https://learn.microsoft.com/ja-jp/power-platform/admin/support-overview#what-hours-are-considered-local-business-hours-for-support)内でのご支援を承っております。
そのため夜間や休日に移行を実施する場合には、営業時間内での技術サポートからの完了報告が叶いませんので、技術サポートからは翌営業日の営業時間に実施結果をご報告いたします。
誠に恐れ入りますが、サポート契約上の制限事項として何卒ご理解を賜りますようお願い申し上げます。

夜間や休日帯の移行に際し、弊社からのご連絡前に結果を確認されたい場合には、移行が完了したと考えられる時間帯以降に、環境がアクセス可能であることや環境の管理モードが無効となっていることをご確認ください。

### 2-4 移行が失敗した場合の対応について
移行が万が一失敗した場合には、以下の対応を行います。

* テナントの移行処理自体が失敗した場合
弊社側で、自動で処理のロールバックを行います。

* ユーザー マッピングが一部ユーザーのライセンス不足などにより失敗した場合
テナント移行の自動ロールバックは行いません。そのため環境の移行自体は完了し、ユーザー マッピングのみが完了していない状態となります。
この場合には、失敗した原因ごとに、ライセンス付与やユーザー マッピングファイルの修正など、発生事象に応じたご対応をお願い申し上げます。

また、特に本番環境の移行が万が一失敗し、これにより広範な業務停止などの重大な影響が発生した場合には、緊急度重大のお問合せを起票いただくことで、事象回避を目的とした 24 時間体制でご支援できる場合がございます。

### 2-5 ユーザー マッピング後のユーザー情報はどのように更新されるか
ユーザー マッピング処理では、環境のユーザー情報のうち、UPN など Azure AD との連携に直接関わる部分の書き換えを行います。
これにより、移行完了後のユーザーは新テナントの Azure AD と同期されるようになります。
従って、その後は通常のユーザー同期機構により、ユーザー情報 (氏名や状態など) の書き換えが追加で行われます。

そのため移行後のユーザー状況が意図したものとなっていない場合、ユーザー同期機構の正常な動きに事象が起因する可能性がございますので、一度 Power Platform 管理センターからユーザーを再度追加するなどし、同期内容をご検証いただけますと幸いです。
よくいただくご質問として、Office 365 ライセンスのみをお持ちのユーザー様について、ユーザー マッピング後に一時的に状態が無効となるケースがございます。
これは当該ユーザーがライセンス要件上、環境に自動で追加されないためでございます。
本挙動は通常環境にユーザーを追加いただく際にも発生する正常なものであり、ユーザーの環境利用には影響いたしません。
・ご参考 : [Dataverse に自動的に追加されないユーザーのカテゴリ](https://learn.microsoft.com/ja-jp/power-platform/admin/create-users#categories-of-users-not-added-automatically-in-dataverse)

---
## 最後に
本記事がテナント移行をご検討いただいている皆様のお役に立てましたら幸いです。
ご不明な点などがございましたら、弊社サポート一同にてお待ち申し上げておりますので、ぜひお気軽にお問合せください。
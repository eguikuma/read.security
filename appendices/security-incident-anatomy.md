---
layout: default
title: セキュリティ事件はなぜ繰り返されるのか
---

# [セキュリティ事件はなぜ繰り返されるのか](#security-incident-anatomy) {#security-incident-anatomy}

## [はじめに](#introduction) {#introduction}

このリポジトリでは、セキュリティの原則を実感するために複数のセキュリティ事件を取り上げました

[03-certificate](../../03-certificate/) では DigiNotar と Symantec の認証局の信頼失墜、[06-application-security](../../06-application-security/) では Next.js ミドルウェアバイパス、[07-supply-chain](../../07-supply-chain/) では Log4Shell、xz-utils バックドア、event-stream 事件を学びました

これらの事件は異なるトピックで異なる文脈で登場しましたが、実は共通するパターンがあります

なぜセキュリティ事件は形を変えながら繰り返されるのでしょうか？

この補足資料では、各トピックに散在する事件を横断的に整理し、事件に共通するパターンと、脆弱性がどのように発見・開示されるかの仕組みを学びます

---

## [このページで学ぶこと](#what-you-will-learn) {#what-you-will-learn}

- セキュリティ事件に共通する構造的パターン
- CVE と脆弱性開示プロセスの仕組み
- 事件から導かれる設計原則

---

## [目次](#table-of-contents) {#table-of-contents}

1. [事件の一覧](#incident-list)
2. [共通パターンの抽出](#common-patterns)
3. [CVE と脆弱性開示](#cve-and-vulnerability-disclosure)
4. [事件から学ぶ設計原則](#design-principles-from-incidents)
5. [まとめ](#summary)
6. [用語集](#glossary)
7. [参考資料](#references)

---

## [事件の一覧](#incident-list) {#incident-list}

このリポジトリで取り上げた事件と、関連するトピックを整理します

{: .labeled}
| 事件 | 年 | 関連トピック | 分類 |
| ------------------------------------ | ------------- | ----------------------- | ---------------- |
| DigiNotar 不正証明書発行 | 2011 年 | 03-certificate | 信頼の悪用 |
| Symantec 認証局の信頼失墜 | 2015〜2017 年 | 03-certificate | 信頼の悪用 |
| Next.js ミドルウェアバイパス | 2025 年 | 06-application-security | 単一レイヤー依存 |
| Log4Shell（CVE-2021-44228） | 2021 年 | 07-supply-chain | 推移的依存関係 |
| xz-utils バックドア（CVE-2024-3094） | 2024 年 | 07-supply-chain | 信頼の悪用 |
| event-stream 事件 | 2018 年 | 07-supply-chain | 信頼の悪用 |

---

## [共通パターンの抽出](#common-patterns) {#common-patterns}

これらの事件を分析すると、3 つの共通パターンが浮かび上がります

### [パターン 1：信頼の悪用](#pattern-trust-abuse) {#pattern-trust-abuse}

<strong>信頼されている存在が、その信頼を裏切る（または悪用される）</strong>

{: .labeled}
| 事件 | 信頼されていた存在 | 何が起きたか |
| ------------ | ---------------------- | -------------------------------------------------------------------------- |
| DigiNotar | 認証局（CA） | 攻撃者が CA のシステムに侵入し、不正な証明書を発行した |
| Symantec | 認証局（CA） | 検証手続きの不備により、ドメイン管理者の承認なしに証明書が発行された |
| xz-utils | オープンソースメンテナ | 攻撃者が数年かけて信頼を獲得し、メンテナ権限を得た後にバックドアを挿入した |
| event-stream | オープンソースメンテナ | メンテナンスを引き継いだ新しい開発者が、悪意のある依存関係を追加した |

03-certificate で学んだ PKI の信頼モデルも、07-supply-chain で学んだ依存関係の信頼も、<strong>「信頼を与える対象が善良であること」を前提</strong>としています

この前提が崩れたとき、信頼モデル全体が機能しなくなります

Certificate Transparency は認証局への信頼を<strong>検証可能な透明性</strong>で補完する仕組みであり、Sigstore は署名の透明性で同じことをソフトウェアサプライチェーンに適用しています

### [パターン 2：単一障害点](#pattern-single-point-of-failure) {#pattern-single-point-of-failure}

<strong>1 つの防御層が突破されると、全体が崩壊する</strong>

{: .labeled}
| 事件 | 単一障害点 | 何が起きたか |
| ---------------------------- | ------------------------------ | ------------------------------------------------------------------- |
| DigiNotar | 認証局のシステムセキュリティ | CA のシステムが侵入されただけで、任意のドメインの証明書が発行された |
| Next.js ミドルウェアバイパス | ミドルウェアによる認証チェック | 特定のヘッダーを設定するだけで認証チェックを回避できた |

06-application-security で学んだ<strong>多層防御（Defense in Depth）</strong>は、この問題への対策です

1 つの防御層が突破されても、次の防御層がブロックする設計にすることで、単一障害点を排除します

### [パターン 3：検知の遅れ](#pattern-delayed-detection) {#pattern-delayed-detection}

<strong>攻撃が発見されるまでに長期間が経過する</strong>

{: .labeled}
| 事件 | 攻撃の期間 | 発見までの経緯 |
| ------------ | ------------------------ | --------------------------------- |
| xz-utils | 約 2 年間の信頼獲得活動 | 偶然の性能調査で発見 |
| event-stream | 数ヶ月 | 被害者による調査で発見 |
| Log4Shell | 脆弱性は 2013 年から存在 | 2021 年に攻撃手法が公開されて発覚 |

Log4Shell の脆弱性は約 8 年間コードに存在していましたが、その間発見されませんでした

xz-utils のバックドアは、Microsoft のエンジニアが SSH 接続の遅延を調査する中で偶然発見されたものです

これらの事例は、<strong>「攻撃が起きていない」ことと「攻撃が検知されていない」ことは異なる</strong>ことを示しています

---

## [CVE と脆弱性開示](#cve-and-vulnerability-disclosure) {#cve-and-vulnerability-disclosure}

### [CVE とは](#what-is-cve) {#what-is-cve}

このリポジトリでは CVE-2021-44228（Log4Shell）や CVE-2024-3094（xz-utils）といった識別子が登場しました

<strong>CVE</strong>（Common Vulnerabilities and Exposures）は、公開されたセキュリティ脆弱性に一意の識別子を付与するシステムです

CVE は米国の MITRE Corporation が管理しており、脆弱性ごとに <strong>CVE-年-番号</strong> の形式で識別子が割り当てられます

CVE の目的は、異なる組織やツール間で<strong>同じ脆弱性を一意に参照</strong>できるようにすることです

「Log4j の脆弱性」と言っても複数の脆弱性がありますが、「CVE-2021-44228」と言えば特定の脆弱性を正確に指定できます

### [脆弱性の深刻度（CVSS）](#cvss) {#cvss}

CVE に登録された脆弱性には、<strong>CVSS</strong>（Common Vulnerability Scoring System）によるスコアが付与されます

CVSS は 0.0 から 10.0 までの数値で脆弱性の深刻度を評価します

{: .labeled}
| スコア範囲 | 深刻度 |
| ---------- | ------ |
| 0.0 | なし |
| 0.1〜3.9 | 低 |
| 4.0〜6.9 | 中 |
| 7.0〜8.9 | 高 |
| 9.0〜10.0 | 緊急 |

Log4Shell（CVE-2021-44228）は CVSS スコア <strong>10.0</strong>（最大値）が付与されました

リモートから認証なしで任意のコードを実行できるという、あらゆる評価基準で最悪の条件を満たしていたためです

### [責任ある開示](#responsible-disclosure) {#responsible-disclosure}

脆弱性を発見した研究者は、その情報をどのように公開すべきでしょうか

<strong>責任ある開示</strong>（Responsible Disclosure、または Coordinated Disclosure）は、脆弱性の発見者がソフトウェアの開発元に<strong>非公開で報告し、修正が完了した後に公開する</strong>プロセスです

{: .labeled}
| ステップ | 説明 |
| -------- | ---------------------------------------------- |
| 1. 報告 | 発見者が開発元に非公開で脆弱性を報告する |
| 2. 確認 | 開発元が脆弱性の存在を確認する |
| 3. 修正 | 開発元がパッチを開発する |
| 4. 公開 | パッチのリリースと同時に脆弱性の詳細を公開する |

修正前に脆弱性を公開すると、パッチが存在しない状態で攻撃者に情報を与えてしまいます

一方、開発元が修正に時間をかけすぎる場合、ユーザーは脆弱性の存在を知らないまま危険にさらされ続けます

このバランスを取るため、一般的には報告から <strong>90 日</strong>を目安に公開するという慣行があります

---

## [事件から学ぶ設計原則](#design-principles-from-incidents) {#design-principles-from-incidents}

メイントピックで学んだ設計原則が、事件のどのパターンへの対策になるかを整理します

### [多層防御（Defense in Depth）](#defense-in-depth) {#defense-in-depth}

06-application-security で学んだ多層防御は、<strong>単一障害点</strong>への対策です

Next.js ミドルウェアバイパスの教訓は、「1 つの防御層だけに頼らない」ことです

{: .labeled}
| レイヤー | 対策の例 |
| ------------------ | ---------------------------------------------------- |
| アプリケーション層 | 入力検証、出力エンコーディング |
| 認証 / 認可層 | ミドルウェアだけでなく、API ハンドラでも権限チェック |
| インフラ層 | ネットワーク制限、WAF |

### [最小権限の原則](#least-privilege-principle) {#least-privilege-principle}

05-access-control で学んだ最小権限の原則は、<strong>信頼の悪用</strong>による被害を限定します

xz-utils 事件では、メンテナ権限が広範であったために、バックドアの挿入が可能でした

権限を必要最小限に制限することで、信頼が裏切られた場合の影響範囲を小さくできます

### [透明性と検証可能性](#transparency-and-verifiability) {#transparency-and-verifiability}

03-certificate で学んだ Certificate Transparency と、07-supply-chain で学んだ Sigstore は、<strong>信頼の悪用</strong>と<strong>検知の遅れ</strong>への対策です

{: .labeled}
| 仕組み | 対策 |
| ------------------------ | -------------------------------------------------------------------- |
| Certificate Transparency | 認証局の発行行為を公開ログに記録し、不正発行を検知可能にする |
| Sigstore（Rekor） | ソフトウェアの署名行為を公開ログに記録し、不正な署名を検知可能にする |

これらの仕組みに共通するのは、<strong>「信頼するが検証する」</strong>というアプローチです

信頼を完全に排除するのではなく、信頼が正しく行使されていることを第三者が検証できるようにします

### [フェイルセーフ](#fail-safe) {#fail-safe}

「安全側に倒す」という設計原則です

障害や攻撃が発生した場合、システムは<strong>安全な状態</strong>に移行すべきです

たとえば、認証チェックが失敗した場合はアクセスを<strong>拒否</strong>するのがフェイルセーフです

認証チェックのエラーをアクセス許可として扱う設計は、Next.js ミドルウェアバイパスのような問題を招きます

---

## [まとめ](#summary) {#summary}

{: .labeled}
| ポイント | 説明 |
| -------------------------------------- | ------------------------------------------------------------------------ |
| セキュリティ事件には共通パターンがある | 信頼の悪用、単一障害点、検知の遅れの 3 つが繰り返し現れる |
| CVE は脆弱性の共通識別子 | 異なる組織やツール間で同じ脆弱性を一意に参照するためのシステム |
| 責任ある開示が被害を最小化する | 修正前の公開は攻撃者を利するため、開発元との協調が重要 |
| 設計原則が事件のパターンに対応する | 多層防御は単一障害点に、最小権限は信頼の悪用に、透明性は検知の遅れに対応 |

---

## [用語集](#glossary) {#glossary}

{: .labeled}
| 用語 | 説明 |
| -------------- | --------------------------------------------------------------------------------------------------------- |
| CVE | Common Vulnerabilities and Exposures の略で、公開されたセキュリティ脆弱性に一意の識別子を付与するシステム |
| CVSS | Common Vulnerability Scoring System の略で、脆弱性の深刻度を 0.0〜10.0 のスコアで評価するシステム |
| 責任ある開示 | 脆弱性の発見者が開発元に非公開で報告し、修正後に公開するプロセス |
| 単一障害点 | 1 つの要素の失敗がシステム全体の障害につながるポイント |
| フェイルセーフ | 障害発生時にシステムが安全な状態に移行する設計原則 |

---

## [参考資料](#references) {#references}

<strong>CVE</strong>

- [CVE - Common Vulnerabilities and Exposures](https://www.cve.org/){:target="\_blank"}
  - CVE の公式サイト

<strong>CVSS</strong>

- [Common Vulnerability Scoring System v3.1](https://www.first.org/cvss/v3.1/specification-document){:target="\_blank"}
  - CVSS の仕様書

<strong>事件</strong>

- [CVE-2021-44228 - Apache Log4j](https://www.cve.org/CVERecord?id=CVE-2021-44228){:target="\_blank"}
  - Log4Shell の CVE レコード

- [CVE-2024-3094 - xz-utils](https://www.cve.org/CVERecord?id=CVE-2024-3094){:target="\_blank"}
  - xz-utils バックドアの CVE レコード

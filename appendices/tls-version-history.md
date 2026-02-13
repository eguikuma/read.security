---
layout: default
title: SSL から TLS 1.3 への道のり
---

# [appendix：SSL から TLS 1.3 への道のり](#tls-version-history) {#tls-version-history}

## [はじめに](#introduction) {#introduction}

[02-tls](../../02-tls/) では、TLS 1.3（RFC 8446）を題材に、暗号化技術の組み合わせが「なぜ安全か」を学びました

02-tls の中で、TLS 1.2 との比較がいくつか登場しました

たとえば、TLS 1.3 では暗号スイートの選択肢が大幅に削減されたこと、ハンドシェイクが 1-RTT に短縮されたことなどです

しかし、SSL から TLS 1.3 に至るまでの全体的な歴史は扱いませんでした

TLS 1.3 の設計は、過去のバージョンで発見された脆弱性への反省から生まれています

「なぜ TLS 1.3 はこのように設計されたのか」を理解するには、過去のバージョンがどのような攻撃を受け、どのように改善されてきたかを知る必要があります

この補足資料では、SSL 2.0 から TLS 1.3 に至るまでの歴史を辿り、各バージョンの設計判断とその背景にある攻撃を学びます

---

## [このページで学ぶこと](#what-you-will-learn) {#what-you-will-learn}

- SSL / TLS の各バージョンがどのような課題を解決したか
- 過去のバージョンに対する代表的な攻撃
- TLS 1.3 の設計が過去の教訓からどう導かれたか

---

## [目次](#table-of-contents) {#table-of-contents}

1. [年表](#timeline)
2. [SSL 2.0（1995 年）](#ssl-2)
3. [SSL 3.0（1996 年）](#ssl-3)
4. [TLS 1.0（1999 年）](#tls-1-0)
5. [TLS 1.1（2006 年）](#tls-1-1)
6. [TLS 1.2（2008 年）](#tls-1-2)
7. [TLS 1.3（2018 年）](#tls-1-3)
8. [バージョンの廃止と移行](#version-deprecation-and-migration)
9. [用語集](#glossary)
10. [参考資料](#references)

---

## [年表](#timeline) {#timeline}

{: .labeled}
| 年 | バージョン | 概要 |
| ------- | ---------- | ------------------------------------------------------ |
| 1994 年 | SSL 2.0 | Netscape が設計。Web の暗号化通信の先駆け |
| 1996 年 | SSL 3.0 | SSL 2.0 の脆弱性を修正した全面的な再設計 |
| 1999 年 | TLS 1.0 | IETF が標準化。SSL 3.0 をベースに改良（RFC 2246） |
| 2006 年 | TLS 1.1 | CBC 攻撃への対策を追加（RFC 4346） |
| 2008 年 | TLS 1.2 | 暗号スイートの柔軟性を大幅に向上（RFC 5246） |
| 2018 年 | TLS 1.3 | ハンドシェイクの再設計、レガシー暗号の排除（RFC 8446） |

---

## [SSL 2.0（1995 年）](#ssl-2) {#ssl-2}

### [背景](#background) {#background}

1990 年代前半、Web ブラウザ Netscape Navigator がインターネット上の通信を暗号化するためのプロトコルとして <strong>SSL</strong>（Secure Sockets Layer）を開発しました

SSL 1.0 は設計段階で重大な欠陥が見つかり公開されませんでした

1995 年に公開された <strong>SSL 2.0</strong> が、実際に使われた最初の SSL バージョンです

### [設計上の問題](#ssl2-design-problems) {#ssl2-design-problems}

SSL 2.0 は Web 暗号化の先駆けでしたが、多くの設計上の問題がありました

{: .labeled}
| 問題 | 説明 |
| ------------------------------------ | ---------------------------------------------------- |
| ハンドシェイクの認証がない | ハンドシェイクメッセージが改ざんされても検出できない |
| 弱い MAC 構造 | MD5 を使った MAC（メッセージ認証コード）が脆弱 |
| ダウングレード攻撃に対する防御がない | 攻撃者が弱い暗号スイートの使用を強制できる |
| 証明書チェーンの検証が不十分 | サーバー認証の仕組みが不完全 |

これらの問題により、SSL 2.0 は比較的短期間で後継バージョンに置き換えられました

---

## [SSL 3.0（1996 年）](#ssl-3) {#ssl-3}

### [改善点](#improvements) {#improvements}

SSL 2.0 の欠陥を受けて、<strong>SSL 3.0</strong> は全面的に再設計されました

{: .labeled}
| SSL 2.0 の問題 | SSL 3.0 での改善 |
| ------------------------------ | ---------------------------------------------------------------------------------------------- |
| ハンドシェイクの改ざん検出なし | ハンドシェイクの最後に Finished メッセージを追加し、全ハンドシェイクメッセージのハッシュを検証 |
| 弱い MAC | HMAC に近い構造を採用（ただし、正式な HMAC 仕様は後に RFC 2104 で定義） |
| ダウングレード攻撃 | ハンドシェイクメッセージのハッシュによる検出 |

SSL 3.0 は 02-tls で学んだ TLS ハンドシェイクの基本構造（ClientHello → ServerHello → 鍵交換 → Finished）を確立しました

### [POODLE 攻撃（2014 年）](#poodle-attack) {#poodle-attack}

SSL 3.0 は約 18 年間使われましたが、2014 年に <strong>POODLE</strong>（Padding Oracle On Downgraded Legacy Encryption）攻撃が発表されました

POODLE は、SSL 3.0 の <strong>CBC モード</strong>のパディング処理の欠陥を利用する攻撃です

01-cryptography で学んだブロック暗号は、固定長のブロック単位でデータを処理します

データがブロック長の倍数でない場合、<strong>パディング</strong>（詰め物）を追加して長さを揃えます

SSL 3.0 ではパディングの内容を厳密に検証していなかったため、攻撃者はパディングを操作することで暗号文から 1 バイトずつ平文を復元できました

POODLE 攻撃により、SSL 3.0 は 2015 年に RFC 7568 で使用が禁止されました

---

## [TLS 1.0（1999 年）](#tls-1-0) {#tls-1-0}

### [SSL から TLS へ](#ssl-to-tls) {#ssl-to-tls}

1999 年、IETF（Internet Engineering Task Force）が SSL 3.0 をベースに標準化したプロトコルが <strong>TLS 1.0</strong>（RFC 2246）です

名称が「SSL」から「TLS」に変わったのは、SSL が Netscape の独自プロトコルだったのに対し、TLS は IETF のオープンな標準であることを明確にするためです

TLS 1.0 は SSL 3.0 と大きく異なるわけではありませんが、以下の改善が行われました

- HMAC の正式な採用（SSL 3.0 の独自 MAC から RFC 2104 準拠の HMAC へ）
- PRF（擬似ランダム関数）の導入による鍵導出の改善

### [BEAST 攻撃（2011 年）](#beast-attack) {#beast-attack}

<strong>BEAST</strong>（Browser Exploit Against SSL/TLS）は、TLS 1.0 の CBC モードの実装上の脆弱性を利用する攻撃です

TLS 1.0 の CBC モードでは、前のレコードの暗号文の最後のブロックを次のレコードの <strong>IV（初期化ベクトル）</strong>として使用していました

01-cryptography で学んだとおり、IV は暗号化のランダム性を確保するための値です

しかし、前のレコードの暗号文は攻撃者にも観測可能であるため、IV が予測可能になります

攻撃者は予測可能な IV を利用して、選択平文攻撃（意図的に送信させた平文と暗号文の関係を利用する攻撃）を実行し、暗号化された Cookie などの値を復元できました

---

## [TLS 1.1（2006 年）](#tls-1-1) {#tls-1-1}

### [BEAST への対策](#beast-countermeasure) {#beast-countermeasure}

<strong>TLS 1.1</strong>（RFC 4346）は、BEAST 攻撃の根本原因であった IV の問題を修正しました

{: .labeled}
| TLS 1.0 | TLS 1.1 |
| -------------------------------------- | ---------------------------------------- |
| 前のレコードの暗号文を IV として再利用 | レコードごとに明示的なランダム IV を使用 |

レコードごとに新しいランダムな IV を使用することで、IV が予測可能になる問題を解消しました

その他の改善は限定的であり、TLS 1.1 は TLS 1.0 からの小規模な改良でした

---

## [TLS 1.2（2008 年）](#tls-1-2) {#tls-1-2}

### [柔軟性の向上](#flexibility-improvement) {#flexibility-improvement}

<strong>TLS 1.2</strong>（RFC 5246）は、暗号の柔軟性を大幅に向上させたバージョンです

{: .labeled}
| 改善点 | 説明 |
| -------------------------- | --------------------------------------------------------------------------------------- |
| ハッシュアルゴリズムの選択 | MD5 / SHA-1 に固定されていた PRF のハッシュアルゴリズムを、暗号スイートごとに選択可能に |
| AEAD の導入 | 01-cryptography で学んだ AEAD（AES-GCM、AES-CCM）を暗号スイートとしてサポート |
| 署名アルゴリズムの拡張 | ハンドシェイク中の署名アルゴリズムを柔軟に選択可能に |

AEAD の導入は特に重要です

CBC モードでは暗号化と MAC を別々に適用するため、パディングオラクル攻撃のような問題が生じました

AEAD では暗号化と認証が統合されているため、この種の攻撃を構造的に防ぎます

### [TLS 1.2 の課題](#tls-1-2-challenges) {#tls-1-2-challenges}

TLS 1.2 は柔軟性が高い反面、<strong>暗号スイートの選択肢が多すぎる</strong>という問題がありました

弱い暗号スイート（RC4、DES、輸出グレード暗号など）もサポートされており、以下のような攻撃を招きました

{: .labeled}
| 攻撃名 | 年 | 概要 |
| -------- | ------- | ---------------------------------------------------------------------------- |
| CRIME | 2012 年 | TLS 圧縮を利用して暗号化された Cookie を復元 |
| Lucky 13 | 2013 年 | CBC モードの MAC 検証タイミングの微小な差を利用 |
| Logjam | 2015 年 | 512 ビットの Diffie-Hellman 輸出グレードパラメータをダウングレード攻撃で強制 |
| FREAK | 2015 年 | 輸出グレードの RSA 暗号をダウングレード攻撃で強制 |

これらの攻撃の多くは、<strong>古い暗号方式のサポートが残っていたこと</strong>が原因です

02-tls で学んだダウングレード攻撃の防御が、TLS 1.3 の設計で特に重視された理由がここにあります

---

## [TLS 1.3（2018 年）](#tls-1-3) {#tls-1-3}

### [過去の教訓からの再設計](#redesign-from-lessons) {#redesign-from-lessons}

<strong>TLS 1.3</strong>（RFC 8446）は、過去のバージョンの問題を踏まえた大幅な再設計です

02-tls で学んだ TLS 1.3 の各設計判断が、どの教訓から導かれたかを見てみましょう

{: .labeled}
| TLS 1.3 の設計 | 背景にある教訓 |
| --------------------------- | ---------------------------------------------------------------------------- |
| 暗号スイートを 5 つに限定 | TLS 1.2 の過剰な選択肢が弱い暗号の使用を招いた |
| CBC モードの廃止、AEAD のみ | POODLE、Lucky 13 など CBC 関連の攻撃が繰り返された |
| 静的 RSA 鍵交換の廃止 | 前方秘匿性のない鍵交換は、秘密鍵の漏洩で過去の通信が全て復号される |
| 圧縮の廃止 | CRIME 攻撃の原因 |
| ハンドシェイクの暗号化 | TLS 1.2 以前ではハンドシェイクの大部分が平文で、証明書の内容が盗聴可能だった |
| 1-RTT ハンドシェイク | 安全性を維持しつつ、パフォーマンスを向上 |

### [削除された暗号方式](#removed-cipher-suites) {#removed-cipher-suites}

TLS 1.3 で削除された暗号方式と、その理由を整理します

{: .labeled}
| 削除された暗号方式 | 理由 |
| ------------------------ | ------------------------------------------------------------------------ |
| RC4 | 統計的バイアスが発見され、暗号として安全ではない |
| DES / 3DES | 鍵長が不十分（DES）、ブロック長が 64 ビットで Sweet32 攻撃の対象（3DES） |
| CBC モードの暗号スイート | パディングオラクル攻撃の温床 |
| 静的 RSA 鍵交換 | 前方秘匿性がない |
| 輸出グレード暗号 | 鍵長が極端に短く、安全ではない |
| MD5 / SHA-1 による署名 | ハッシュ関数の衝突耐性が不十分 |

「削除する」という判断は、「追加する」よりも難しいものです

後方互換性を壊す可能性があるためです

しかし、TLS 1.3 は安全性を最優先とし、<strong>弱い暗号を残すリスクよりも、削除するコストを選びました</strong>

---

## [バージョンの廃止と移行](#version-deprecation-and-migration) {#version-deprecation-and-migration}

各バージョンの廃止は段階的に進められました

{: .labeled}
| バージョン | 廃止の RFC / 決定 | 廃止年 |
| ---------- | ------------------- | ------- |
| SSL 2.0 | RFC 6176 で使用禁止 | 2011 年 |
| SSL 3.0 | RFC 7568 で使用禁止 | 2015 年 |
| TLS 1.0 | RFC 8996 で使用禁止 | 2021 年 |
| TLS 1.1 | RFC 8996 で使用禁止 | 2021 年 |

TLS 1.0 と TLS 1.1 は同時に廃止されました

主要なブラウザ（Chrome、Firefox、Safari、Edge）は、2020 年にこれらのバージョンのサポートを終了しています

SSL / TLS の歴史は、暗号プロトコルの設計が<strong>攻撃と防御の繰り返し</strong>で進化してきたことを示しています

新しい攻撃手法が発見されるたびに、プロトコルは修正され、より安全なバージョンに進化しました

TLS 1.3 はこの進化の最新地点ですが、暗号の歴史が示すように、これが最終形ではないかもしれません

---

## [用語集](#glossary) {#glossary}

{: .labeled}
| 用語 | 説明 |
| ---------------------- | ---------------------------------------------------------------------------------------------------------- |
| SSL | Secure Sockets Layer の略で、Netscape が開発した暗号化通信プロトコル |
| TLS | Transport Layer Security の略で、IETF が SSL を標準化した暗号化通信プロトコル |
| CBC モード | Cipher Block Chaining の略で、前のブロックの暗号文を次のブロックの暗号化に利用するブロック暗号の動作モード |
| パディング | ブロック暗号でデータ長をブロック長の倍数に揃えるために追加するデータ |
| パディングオラクル攻撃 | サーバーのパディング検証のエラー応答を利用して暗号文を解読する攻撃 |
| 輸出グレード暗号 | 米国の暗号輸出規制に対応するために鍵長を制限した弱い暗号方式 |
| POODLE | SSL 3.0 の CBC パディング処理の欠陥を利用する攻撃 |
| BEAST | TLS 1.0 の CBC モードの予測可能な IV を利用する攻撃 |
| Logjam | 輸出グレードの Diffie-Hellman パラメータへのダウングレードを強制する攻撃 |

---

## [参考資料](#references) {#references}

<strong>TLS 仕様</strong>

- [RFC 8446 - The Transport Layer Security (TLS) Protocol Version 1.3](https://datatracker.ietf.org/doc/html/rfc8446){:target="\_blank"}
  - TLS 1.3 の仕様

- [RFC 5246 - The Transport Layer Security (TLS) Protocol Version 1.2](https://datatracker.ietf.org/doc/html/rfc5246){:target="\_blank"}
  - TLS 1.2 の仕様

<strong>廃止</strong>

- [RFC 8996 - Deprecating TLS 1.0 and TLS 1.1](https://datatracker.ietf.org/doc/html/rfc8996){:target="\_blank"}
  - TLS 1.0 と TLS 1.1 の使用禁止

- [RFC 7568 - Deprecating Secure Sockets Layer Version 3.0](https://datatracker.ietf.org/doc/html/rfc7568){:target="\_blank"}
  - SSL 3.0 の使用禁止

<strong>攻撃</strong>

- [POODLE - This POODLE Bites: Exploiting The SSL 3.0 Fallback](https://www.openssl.org/~bodo/ssl-poodle.pdf){:target="\_blank"}
  - SSL 3.0 に対する POODLE 攻撃の論文

- [Logjam - Imperfect Forward Secrecy: How Diffie-Hellman Fails in Practice](https://weakdh.org/){:target="\_blank"}
  - Diffie-Hellman の弱いパラメータに対する攻撃

---
layout: default
title: なぜセキュリティに「良い乱数」が必要なのか
---

# [なぜセキュリティに「良い乱数」が必要なのか](#why-good-random-numbers) {#why-good-random-numbers}

## [はじめに](#introduction) {#introduction}

[01-cryptography](../../01-cryptography/) では、対称暗号の鍵、Diffie-Hellman 鍵交換のエフェメラル鍵、ハッシュ関数に入力するソルトなど、暗号化の各所で「ランダムな値」が登場しました

[02-tls](../../02-tls/) では、TLS ハンドシェイクでクライアントとサーバーがそれぞれランダムな値を生成することを学びました

[04-authentication](../../04-authentication/) では、セッション ID やトークンに「予測不可能なランダムな文字列」が必要であることを学びました

これらの場面で繰り返し登場する「ランダム」という言葉ですが、乱数そのものの仕組みについてはまだ触れていません

コンピュータは決定的な機械であり、本質的に「ランダム」な動作はしません

では、暗号に必要な乱数はどのように生成されているのでしょうか？

そして、なぜ「良い乱数」が暗号の安全性にとって決定的に重要なのでしょうか？

この補足資料では、暗号における乱数の役割と、乱数の品質がセキュリティに与える影響を学びます

---

## [このページで学ぶこと](#what-you-will-learn) {#what-you-will-learn}

- 暗号のどこで乱数が使われているか
- 擬似乱数生成器（PRNG）と暗号論的擬似乱数生成器（CSPRNG）の違い
- エントロピーとは何か
- OS が提供する乱数源の仕組み
- 乱数の品質が低いと何が起きるか

---

## [目次](#table-of-contents) {#table-of-contents}

1. [暗号と乱数の関係](#cryptography-and-random-number-relationship)
2. [擬似乱数生成器（PRNG）](#prng)
3. [暗号論的擬似乱数生成器（CSPRNG）](#csprng)
4. [エントロピー](#entropy)
5. [OS が提供する乱数源](#os-random-sources)
6. [乱数の品質が低いとき何が起きるか](#low-quality-random-consequences)
7. [まとめ](#summary)
8. [用語集](#glossary)
9. [参考資料](#references)

---

## [暗号と乱数の関係](#cryptography-and-random-number-relationship) {#cryptography-and-random-number-relationship}

暗号の安全性は、鍵やパラメータが攻撃者に<strong>予測できない</strong>ことに依存しています

以下は、このリポジトリのメイントピックで登場した「乱数が使われる場面」のまとめです

{: .labeled}
| 場面 | 使われ方 | 関連トピック |
| ------------------------------------------------ | ---------------------------------------------------------- | ----------------------- |
| 対称暗号の鍵生成 | ランダムなビット列を鍵として使用 | 01-cryptography |
| ナンス / IV（初期化ベクトル） | 同じ鍵でも異なる暗号文を生成するためのランダムな値 | 01-cryptography |
| Diffie-Hellman のエフェメラル秘密鍵 | 鍵交換のためのランダムな秘密値 | 01-cryptography、02-tls |
| TLS ハンドシェイクの Client Hello / Server Hello | ランダムな 32 バイトの値 | 02-tls |
| ソルト（パスワードハッシュ） | 同じパスワードでも異なるハッシュ値にするためのランダムな値 | 04-authentication |
| セッション ID | 予測不可能なセッション識別子 | 04-authentication |
| CSRF トークン | リクエストの正当性を検証するためのランダムなトークン | 06-application-security |

これらの全てに共通するのは、<strong>攻撃者に予測されたら安全性が崩壊する</strong>ということです

たとえば、TLS ハンドシェイクで使われるエフェメラル秘密鍵が予測できれば、共有秘密を計算でき、通信を復号できてしまいます

セッション ID が予測できれば、他人のセッションを乗っ取ることができます

暗号の安全性は、アルゴリズムの強度と<strong>乱数の品質</strong>の両方に依存しているのです

---

## [擬似乱数生成器（PRNG）](#prng) {#prng}

### [コンピュータは「真のランダム」を作れない](#computers-cannot-be-truly-random) {#computers-cannot-be-truly-random}

コンピュータは<strong>決定的（deterministic）</strong>な機械です

同じ入力に対して常に同じ出力を返します

したがって、ソフトウェアだけで「真にランダムな」数を生成することはできません

ソフトウェアが生成するのは<strong>擬似乱数（pseudo-random numbers）</strong>、つまり「ランダムに見えるが、実際には決定的なアルゴリズムで生成された数」です

### [PRNG の仕組み](#prng-mechanism) {#prng-mechanism}

<strong>PRNG</strong>（Pseudo-Random Number Generator、擬似乱数生成器）は、<strong>シード（seed）</strong>と呼ばれる初期値を入力として受け取り、そのシードから決定的に数列を生成するアルゴリズムです

```
シード → [PRNG アルゴリズム] → 数列（ランダムに見える）
```

同じシードを与えれば、常に同じ数列が生成されます

統計的な検定（均一性、独立性など）に合格する PRNG は、シミュレーションやゲームなどの用途には十分です

### [PRNG が暗号に使えない理由](#why-prng-not-for-cryptography) {#why-prng-not-for-cryptography}

一般的な PRNG は、出力列の一部から<strong>内部状態を逆算</strong>できることがあります

たとえば、多くのプログラミング言語の標準ライブラリに含まれる<strong>線形合同法</strong>は、以下の式で次の値を計算します

```
次の値 = (a × 現在の値 + c) mod m
```

a、c、m は固定の定数であり、出力された値をいくつか観測すれば、次の値を予測できます

つまり、一般的な PRNG の出力から将来の出力を予測でき、暗号の鍵やトークンの生成に使うと攻撃者に値を推測されてしまいます

---

## [暗号論的擬似乱数生成器（CSPRNG）](#csprng) {#csprng}

### [CSPRNG とは](#what-is-csprng) {#what-is-csprng}

<strong>CSPRNG</strong>（Cryptographically Secure Pseudo-Random Number Generator、暗号論的擬似乱数生成器）は、暗号用途に使える擬似乱数生成器です

CSPRNG は、一般的な PRNG よりも強い性質を持ちます

{: .labeled}
| 性質 | PRNG | CSPRNG |
| ------------------ | -------------------------------------------- | -------------------------------------------------------- |
| 統計的なランダム性 | あり | あり |
| 予測不可能性 | なし（出力から次の値を予測可能な場合がある） | あり（出力から次の値を予測するのが計算量的に困難） |
| 逆推測耐性 | なし（内部状態を逆算できる場合がある） | あり（内部状態が漏洩しても過去の出力を逆算するのが困難） |

### [CSPRNG が満たすべき条件](#csprng-requirements) {#csprng-requirements}

CSPRNG は以下の 2 つの条件を満たします

<strong>1. 予測不可能性（Next-bit test）</strong>

出力列の一部を観測しても、次のビットが 0 か 1 かを 50% より有意に高い確率で予測できないこと

<strong>2. 逆推測耐性（State compromise extension）</strong>

内部状態が攻撃者に漏洩した場合でも、それ以前に生成された出力を復元できないこと

これらの条件により、CSPRNG の出力は暗号の鍵、トークン、ナンスなどの安全な生成に使用できます

---

## [エントロピー](#entropy) {#entropy}

### [エントロピーとは](#what-is-entropy) {#what-is-entropy}

CSPRNG は決定的なアルゴリズムであるため、良い出力を生成するには<strong>良い入力（シード）</strong>が必要です

このシードの「良さ」を測る指標が<strong>エントロピー</strong>です

エントロピーとは、情報理論における<strong>不確実性の量</strong>を表す概念です

暗号の文脈では、<strong>攻撃者にとっての予測の困難さ</strong>を意味します

たとえば、コインを 1 回投げた結果は 1 ビットのエントロピーを持ちます（2 通りの等確率な結果）

6 面のサイコロを 1 回投げた結果は約 2.6 ビットのエントロピーを持ちます

### [エントロピー源](#entropy-sources) {#entropy-sources}

コンピュータがエントロピーを得るには、<strong>物理的な現象</strong>からの入力が必要です

決定的なアルゴリズムからはエントロピーは生まれないため、ハードウェアや環境のノイズに頼ります

{: .labeled}
| エントロピー源 | 説明 |
| ------------------------------------ | ---------------------------------------------------------------------------- |
| ハードウェア乱数生成器 | CPU 内蔵の乱数生成器（Intel の RDRAND 命令など）が電気的ノイズから乱数を生成 |
| デバイスの割り込みタイミング | キーボード入力、マウス移動、ディスク I/O の完了タイミングの微小なゆらぎ |
| ネットワークパケットの到着タイミング | パケットが到着する時刻の微小なゆらぎ |
| ジッター | CPU クロックの微小な変動 |

これらの物理的な入力は予測が困難であり、エントロピーの源として利用できます

---

## [OS が提供する乱数源](#os-random-sources) {#os-random-sources}

### [エントロピープール](#entropy-pool) {#entropy-pool}

OS のカーネルは、上記のエントロピー源から集めた乱数を<strong>エントロピープール</strong>に蓄積します

アプリケーションが暗号用の乱数を必要とするとき、OS はエントロピープールからデータを取り出し、CSPRNG を通して出力します

```
[ハードウェアノイズ]  ─┐
[割り込みタイミング]  ─┤
[ネットワークジッター] ─┤→ [エントロピープール] → [CSPRNG] → 暗号用乱数
[CPU ジッター]        ─┘
```

### [Linux の乱数インターフェース](#linux-random-interface) {#linux-random-interface}

Linux カーネルは、暗号用の乱数を取得するためのインターフェースを提供しています

<strong>/dev/urandom</strong>

ブロックせずに暗号用の乱数を返すデバイスファイルです

カーネル内の CSPRNG（ChaCha20 ベース）が初期化された後は、十分な品質の乱数を生成し続けます

<strong>getrandom() システムコール</strong>

Linux 3.17 で追加されたシステムコールで、CSPRNG が初期化されるまでブロックし、初期化後はブロックせずに乱数を返します

/dev/urandom は OS 起動直後の CSPRNG 初期化前にもデータを返してしまうという問題がありましたが、getrandom() はこの問題を解決しています

暗号用の乱数が必要な場面では、プログラミング言語やフレームワークが提供する暗号用乱数 API を使用することが推奨されます

これらの API は内部で OS の CSPRNG を使用しています

---

## [乱数の品質が低いとき何が起きるか](#low-quality-random-consequences) {#low-quality-random-consequences}

乱数の品質が暗号の安全性を決定的に左右することを、実際の事例で見てみましょう

### [Debian OpenSSL の事例（2008 年）](#debian-openssl-case) {#debian-openssl-case}

2006 年、Debian Linux のメンテナが OpenSSL のソースコードを修正しました

コード分析ツール（Valgrind）が「未初期化メモリの使用」を警告したため、該当のコードを削除したのです

しかし、削除されたコードは OpenSSL の CSPRNG にエントロピーを供給する重要な部分でした

この修正により、OpenSSL の CSPRNG のシードが<strong>プロセス ID のみ</strong>に依存するようになりました

Linux のプロセス ID は通常 32768 までの値を取るため、生成される鍵の候補は<strong>約 32768 通り</strong>しかありませんでした

01-cryptography で学んだとおり、AES-128 の鍵空間は 2 の 128 乗です

それが約 32768 通り（2 の 15 乗）にまで縮小されたのです

この脆弱性は 2 年間気づかれず、その間に生成された全ての TLS 証明書、SSH 鍵、その他の暗号鍵が影響を受けました

影響を受けた鍵は全て再生成する必要がありました

### [教訓](#lessons-learned) {#lessons-learned}

Debian OpenSSL の事例は、以下の教訓を示しています

- 暗号アルゴリズムがどれほど強力でも、<strong>鍵がランダムでなければ意味がない</strong>
- エントロピー源の除去は、暗号システム全体の安全性を根底から崩壊させる
- 暗号ライブラリの内部実装に対する変更は、暗号の専門知識なしには行うべきではない

---

## [まとめ](#summary) {#summary}

{: .labeled}
| ポイント | 説明 |
| -------------------------------- | ----------------------------------------------------------------------------------------- |
| 暗号の安全性は乱数に依存する | 鍵、ナンス、ソルト、トークンなど、暗号システムの多くの要素に予測不可能な乱数が必要 |
| 一般的な PRNG は暗号には使えない | 出力から次の値を予測できる可能性があるため、暗号用途には CSPRNG が必要 |
| CSPRNG にはエントロピーが必要 | 良い乱数を出力するには、ハードウェアノイズなどの物理的エントロピー源が不可欠 |
| OS が乱数源を提供する | Linux の /dev/urandom や getrandom() は、カーネル内の CSPRNG を通じて暗号用乱数を提供する |
| 乱数の品質低下は壊滅的 | Debian OpenSSL の事例のように、エントロピー不足は暗号システム全体の安全性を崩壊させる |

---

## [用語集](#glossary) {#glossary}

{: .labeled}
| 用語 | 説明 |
| --------------------------------- | ---------------------------------------------------------------------------------------------- |
| 擬似乱数（pseudo-random numbers） | ランダムに見えるが、決定的なアルゴリズムで生成された数列 |
| PRNG | Pseudo-Random Number Generator の略で、シードから決定的に数列を生成する擬似乱数生成器 |
| CSPRNG | Cryptographically Secure Pseudo-Random Number Generator の略で、暗号用途に使える擬似乱数生成器 |
| シード | 擬似乱数生成器に与える初期値 |
| エントロピー | 情報理論における不確実性の量で、暗号の文脈では攻撃者にとっての予測困難さを表す |
| エントロピープール | OS カーネルが物理的エントロピー源から集めた乱数を蓄積する領域 |
| 線形合同法 | 数式で次の値を計算する単純な擬似乱数生成アルゴリズム（暗号には不適切） |

---

## [参考資料](#references) {#references}

<strong>Linux 乱数</strong>

- [random(7) - Linux manual page](https://man7.org/linux/man-pages/man7/random.7.html){:target="\_blank"}
  - Linux カーネルの乱数生成器の説明

- [getrandom(2) - Linux manual page](https://man7.org/linux/man-pages/man2/getrandom.2.html){:target="\_blank"}
  - getrandom() システムコールの仕様

<strong>NIST 推奨</strong>

- [NIST SP 800-90A - Recommendation for Random Number Generation Using Deterministic Random Bit Generators](https://csrc.nist.gov/pubs/sp/800/90/a/r1/final){:target="\_blank"}
  - NIST による暗号用乱数生成の推奨事項

<strong>Debian OpenSSL</strong>

- [DSA-1571 - openssl -- predictable random number generator](https://www.debian.org/security/2008/dsa-1571){:target="\_blank"}
  - Debian OpenSSL の脆弱性に関する公式アドバイザリ

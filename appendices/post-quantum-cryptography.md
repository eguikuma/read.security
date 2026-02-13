---
layout: default
title: 量子コンピュータは暗号を壊すのか
---

# [量子コンピュータは暗号を壊すのか](#post-quantum-cryptography) {#post-quantum-cryptography}

## [はじめに](#introduction) {#introduction}

[01-cryptography](../../01-cryptography/) では、RSA が「大きな数の素因数分解の困難さ」に、楕円曲線暗号が「楕円曲線上の離散対数問題の困難さ」に安全性を依存していることを学びました

[02-tls](../../02-tls/) では、TLS 1.3 が ECDHE（楕円曲線 Diffie-Hellman）を鍵交換に使用していることを学びました

[03-certificate](../../03-certificate/) では、証明書のデジタル署名が公開鍵暗号に依存していることを学びました

これらの暗号技術は、現在のコンピュータでは素因数分解や離散対数問題を効率的に解けないことを前提としています

しかし、<strong>量子コンピュータ</strong>はこの前提を覆す可能性があります

この補足資料では、量子コンピュータが暗号に与える影響と、その脅威に備えるための<strong>耐量子暗号（Post-Quantum Cryptography）</strong>について学びます

数学的な証明は扱いませんが、「なぜ脅威なのか」「どう対応するのか」の直感的な理解を目指します

---

## [このページで学ぶこと](#what-you-will-learn) {#what-you-will-learn}

- 量子コンピュータがなぜ現在の暗号を脅かすのか
- 対称暗号・ハッシュ関数への影響と非対称暗号への影響の違い
- 「Harvest Now, Decrypt Later」の脅威
- 耐量子暗号（PQC）とは何か

---

## [目次](#table-of-contents) {#table-of-contents}

1. [量子コンピュータと暗号の関係](#quantum-computer-and-cryptography-relationship)
2. [非対称暗号への脅威](#asymmetric-cryptography-threat)
3. [対称暗号とハッシュ関数への影響](#symmetric-and-hash-impact)
4. [Harvest Now, Decrypt Later](#harvest-now-decrypt-later)
5. [耐量子暗号（Post-Quantum Cryptography）](#pqc)
6. [TLS と PKI への影響](#tls-and-pki-impact)
7. [まとめ](#summary)
8. [用語集](#glossary)
9. [参考資料](#references)

---

## [量子コンピュータと暗号の関係](#quantum-computer-and-cryptography-relationship) {#quantum-computer-and-cryptography-relationship}

### [従来のコンピュータと量子コンピュータ](#classical-vs-quantum-computer) {#classical-vs-quantum-computer}

従来のコンピュータはデータを<strong>ビット</strong>（0 または 1）で表現します

量子コンピュータはデータを<strong>量子ビット（qubit）</strong>で表現します

量子ビットは 0 と 1 の<strong>重ね合わせ</strong>の状態を取ることができ、これにより特定の種類の計算を従来のコンピュータよりも指数的に高速に実行できる可能性があります

全ての計算が高速になるわけではありませんが、暗号に関わる特定の数学的問題が量子コンピュータの得意分野に含まれています

### [暗号への影響の概要](#cryptography-impact-overview) {#cryptography-impact-overview}

量子コンピュータの暗号への影響は、暗号の種類によって大きく異なります

{: .labeled}
| 暗号の種類 | 影響 | 対策 |
| ------------------------------------------ | --------------------------------------------------------------------------------- | ------------------------------------------ |
| 非対称暗号（RSA、楕円曲線暗号、DH 鍵交換） | <strong>壊滅的</strong>：安全性の根拠となる問題が量子コンピュータで効率的に解ける | 新しい数学的問題に基づく暗号への移行が必要 |
| 対称暗号（AES） | <strong>限定的</strong>：安全性は低下するが、鍵長を倍にすることで対応可能 | AES-256 への移行で十分 |
| ハッシュ関数（SHA-256） | <strong>限定的</strong>：衝突耐性が低下するが、出力長を増やすことで対応可能 | SHA-256 以上で十分 |

---

## [非対称暗号への脅威](#asymmetric-cryptography-threat) {#asymmetric-cryptography-threat}

### [Shor のアルゴリズム](#shor-algorithm) {#shor-algorithm}

1994 年、数学者 <strong>Peter Shor</strong> が量子コンピュータ上で動作するアルゴリズムを発表しました

<strong>Shor のアルゴリズム</strong>は、整数の<strong>素因数分解</strong>と<strong>離散対数問題</strong>を、従来のコンピュータでは不可能な速度で解くことができます

01-cryptography で学んだ暗号アルゴリズムへの影響を見てみましょう

{: .labeled}
| アルゴリズム | 安全性の根拠 | 量子コンピュータの影響 |
| --------------------- | ------------------------------ | --------------------------------------------- |
| RSA | 大きな数の素因数分解が困難 | Shor のアルゴリズムで効率的に素因数分解が可能 |
| 楕円曲線暗号 | 楕円曲線上の離散対数問題が困難 | Shor のアルゴリズムで効率的に解ける |
| Diffie-Hellman 鍵交換 | 離散対数問題が困難 | Shor のアルゴリズムで効率的に解ける |

つまり、十分に大きな量子コンピュータが実現すれば、RSA の公開鍵から秘密鍵を計算したり、ECDHE の鍵交換を傍受して共有秘密を計算したりすることが可能になります

### [直感的な理解](#intuitive-understanding) {#intuitive-understanding}

なぜ量子コンピュータが素因数分解を高速に行えるのかを、厳密さを犠牲にして直感的に説明します

従来のコンピュータで大きな数を素因数分解するには、候補を順番に試す必要があります

1000 桁の数の素因数分解は、既知の最良のアルゴリズムでも天文学的な時間がかかります

量子コンピュータは量子ビットの重ね合わせを利用して、多数の候補を「並列的に」評価する性質を持っています

Shor のアルゴリズムはこの性質を巧みに利用し、素因数分解に関連する周期性を効率的に発見します

この周期から素因数を導出できるため、従来のコンピュータでは不可能だった速度で素因数分解が完了します

---

## [対称暗号とハッシュ関数への影響](#symmetric-and-hash-impact) {#symmetric-and-hash-impact}

### [Grover のアルゴリズム](#grover-algorithm) {#grover-algorithm}

1996 年、<strong>Lov Grover</strong> が量子コンピュータ上での探索アルゴリズムを発表しました

<strong>Grover のアルゴリズム</strong>は、ソート されていないデータベースからの検索を、従来の計算量の<strong>平方根</strong>で実行できます

対称暗号に対する影響は、鍵の全探索（ブルートフォース）が高速化されることです

{: .labeled}
| 暗号 | 従来の全探索 | Grover による全探索 | 実質的な安全性 |
| ------- | ------------ | ------------------- | --------------------------- |
| AES-128 | 2 の 128 乗 | 2 の 64 乗 | 128 ビット → 64 ビット相当 |
| AES-256 | 2 の 256 乗 | 2 の 128 乗 | 256 ビット → 128 ビット相当 |

Grover のアルゴリズムにより、AES-128 の安全性は 64 ビット相当に低下しますが、<strong>AES-256 は 128 ビット相当の安全性を維持</strong>します

128 ビットの安全性は現在の基準で十分であるため、対称暗号は鍵長を倍にすることで量子コンピュータに対応できます

### [ハッシュ関数への影響](#hash-function-impact) {#hash-function-impact}

ハッシュ関数の衝突探索にも Grover のアルゴリズムが適用できます

{: .labeled}
| ハッシュ関数 | 従来の衝突探索 | 量子コンピュータでの衝突探索 |
| ------------ | -------------- | ---------------------------- |
| SHA-256 | 2 の 128 乗 | 2 の 85 乗程度 |

SHA-256 は量子コンピュータに対しても十分な安全性を持つと考えられています

---

## [Harvest Now, Decrypt Later](#harvest-now-decrypt-later) {#harvest-now-decrypt-later}

量子コンピュータはまだ暗号を破るのに十分な性能を持っていません

しかし、<strong>「Harvest Now, Decrypt Later」</strong>（今収集して、後で復号する）という脅威が存在します

攻撃者が<strong>今の時点で暗号化された通信を傍受・保存</strong>し、将来量子コンピュータが実用化された時点で<strong>過去の通信を復号する</strong>というシナリオです

02-tls で学んだ<strong>前方秘匿性</strong>は、長期的な秘密鍵の漏洩に対する防御でした

しかし、前方秘匿性はエフェメラル鍵（ECDHE）の安全性に依存しています

量子コンピュータが ECDHE を破れるなら、前方秘匿性も失われます

Harvest Now, Decrypt Later の脅威は、<strong>今日の通信のセキュリティが、未来のコンピュータの能力に依存している</strong>ことを意味します

政府の機密情報、医療記録、金融データなど、数十年にわたって機密性を維持する必要があるデータにとって、この脅威は現実的です

このため、量子コンピュータが実用化される<strong>前に</strong>、耐量子暗号への移行を開始する必要があります

---

## [耐量子暗号（Post-Quantum Cryptography）](#pqc) {#pqc}

### [PQC とは](#what-is-pqc) {#what-is-pqc}

<strong>耐量子暗号</strong>（PQC：Post-Quantum Cryptography）は、量子コンピュータでも効率的に解けない数学的問題に基づく暗号アルゴリズムです

重要な点は、PQC は<strong>従来のコンピュータ上で動作する</strong>ことです

量子コンピュータを使って暗号化するのではなく、量子コンピュータによる攻撃に耐えられる暗号を従来のコンピュータで実装します

### [NIST の PQC 標準化](#nist-pqc-standardization) {#nist-pqc-standardization}

[cryptography-history](../cryptography-history/) で学んだ AES の公開コンペと同様に、NIST は 2016 年に <strong>PQC 標準化プロジェクト</strong>を開始しました

世界中の研究者から提出された候補を数年かけて評価し、2024 年に最初の標準が公開されました

{: .labeled}
| 標準 | アルゴリズム | 用途 | 基盤となる数学的問題 |
| -------- | ------------------------------- | ---------------------- | -------------------- |
| FIPS 203 | ML-KEM（旧 CRYSTALS-Kyber） | 鍵カプセル化（鍵交換） | 格子問題 |
| FIPS 204 | ML-DSA（旧 CRYSTALS-Dilithium） | デジタル署名 | 格子問題 |
| FIPS 205 | SLH-DSA（旧 SPHINCS+） | デジタル署名 | ハッシュベース |

### [格子問題の直感的な理解](#lattice-problem) {#lattice-problem}

ML-KEM と ML-DSA の安全性は<strong>格子問題</strong>（lattice problem）に基づいています

格子とは、空間内に規則的に並んだ点の集合です

2 次元の格子を想像すると、方眼紙の交点のようなものです

格子問題の 1 つである<strong>最短ベクトル問題（SVP）</strong>は、「格子の中で原点に最も近い（ゼロでない）点を見つける」問題です

2 次元ではこの問題は簡単ですが、数百次元以上の格子になると、従来のコンピュータでも量子コンピュータでも効率的に解く方法が知られていません

RSA の安全性が「素因数分解は難しい」に依存しているように、PQC の安全性は「高次元の格子問題は難しい」に依存しています

そして、格子問題は Shor のアルゴリズムでは解けないため、量子コンピュータに対しても安全です

---

## [TLS と PKI への影響](#tls-and-pki-impact) {#tls-and-pki-impact}

### [TLS への影響](#tls-impact) {#tls-impact}

02-tls で学んだ TLS 1.3 のハンドシェイクでは、鍵交換に ECDHE、サーバー認証にデジタル署名が使われています

量子コンピュータの脅威に対応するために、TLS の鍵交換と署名アルゴリズムを PQC に移行する必要があります

移行は段階的に進められています

<strong>ハイブリッド方式</strong>は、従来のアルゴリズム（ECDHE）と PQC アルゴリズム（ML-KEM）を組み合わせる方法です

両方のアルゴリズムで鍵交換を行い、どちらか一方が破られても安全性が維持されます

この方式により、PQC アルゴリズムに未知の弱点があった場合でも、従来のアルゴリズムが保険として機能します

### [PKI への影響](#pki-impact) {#pki-impact}

03-certificate で学んだ PKI では、証明書の署名に RSA や ECDSA が使われています

これらも PQC アルゴリズム（ML-DSA など）に移行する必要があります

ただし、PQC の署名と公開鍵は従来のアルゴリズムよりもサイズが大きく、証明書のサイズが増加します

{: .labeled}
| アルゴリズム | 公開鍵のサイズ | 署名のサイズ |
| -------------- | -------------- | ------------ |
| RSA-2048 | 256 バイト | 256 バイト |
| ECDSA（P-256） | 64 バイト | 64 バイト |
| ML-DSA-65 | 1,952 バイト | 3,309 バイト |

証明書のサイズ増加は TLS ハンドシェイクのデータ量に影響するため、移行にはパフォーマンスへの影響も考慮する必要があります

---

## [まとめ](#summary) {#summary}

{: .labeled}
| ポイント | 説明 |
| ---------------------------------------------- | ------------------------------------------------------------------------ |
| 量子コンピュータは非対称暗号を壊滅的に脅かす | Shor のアルゴリズムで RSA、楕円曲線暗号、DH 鍵交換の安全性の根拠が崩れる |
| 対称暗号とハッシュ関数への影響は限定的 | Grover のアルゴリズムで安全性が半減するが、鍵長を倍にすることで対応可能 |
| Harvest Now, Decrypt Later が現実的な脅威 | 今の暗号化通信が将来の量子コンピュータで復号される可能性がある |
| PQC は量子コンピュータでも解けない問題に基づく | 格子問題やハッシュベースの暗号が NIST で標準化された |
| TLS と PKI は段階的に PQC に移行する | ハイブリッド方式で従来のアルゴリズムと PQC を併用しながら移行 |

---

## [用語集](#glossary) {#glossary}

{: .labeled}
| 用語 | 説明 |
| -------------------------- | --------------------------------------------------------------------------------------------------------- |
| 量子ビット（qubit） | 量子コンピュータのデータの基本単位で、0 と 1 の重ね合わせ状態を取れる |
| Shor のアルゴリズム | 量子コンピュータ上で素因数分解と離散対数問題を効率的に解くアルゴリズム |
| Grover のアルゴリズム | 量子コンピュータ上で未整列データベースの検索を平方根の計算量で実行するアルゴリズム |
| Harvest Now, Decrypt Later | 暗号化通信を今保存し、将来の量子コンピュータで復号する攻撃シナリオ |
| 耐量子暗号（PQC） | 量子コンピュータでも解けない問題に基づく暗号アルゴリズムの総称 |
| 格子問題 | 高次元の格子上の最短ベクトルなどを求める問題で、PQC の安全性の基盤 |
| ML-KEM | Module-Lattice-Based Key-Encapsulation Mechanism の略で、NIST が標準化した PQC の鍵カプセル化アルゴリズム |
| ML-DSA | Module-Lattice-Based Digital Signature Algorithm の略で、NIST が標準化した PQC のデジタル署名アルゴリズム |
| ハイブリッド方式 | 従来のアルゴリズムと PQC アルゴリズムを組み合わせて使用する移行戦略 |

---

## [参考資料](#references) {#references}

<strong>NIST PQC 標準</strong>

- [FIPS 203 - Module-Lattice-Based Key-Encapsulation Mechanism Standard](https://csrc.nist.gov/pubs/fips/203/final){:target="\_blank"}
  - ML-KEM（旧 CRYSTALS-Kyber）の仕様

- [FIPS 204 - Module-Lattice-Based Digital Signature Standard](https://csrc.nist.gov/pubs/fips/204/final){:target="\_blank"}
  - ML-DSA（旧 CRYSTALS-Dilithium）の仕様

<strong>NIST PQC プロジェクト</strong>

- [Post-Quantum Cryptography - NIST](https://csrc.nist.gov/projects/post-quantum-cryptography){:target="\_blank"}
  - NIST の耐量子暗号標準化プロジェクトのページ

<strong>量子コンピュータと暗号</strong>

- [NIST IR 8105 - Report on Post-Quantum Cryptography](https://csrc.nist.gov/pubs/ir/8105/final){:target="\_blank"}
  - NIST による耐量子暗号の必要性に関する報告書

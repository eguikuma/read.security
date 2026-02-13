---
layout: default
title: 暗号技術の歴史
---

# [appendix：暗号技術の歴史](#cryptography-history) {#cryptography-history}

## [はじめに](#introduction) {#introduction}

[01-cryptography](../../01-cryptography/) では、AES が「2001 年に NIST が公開コンペで選定した」こと、RSA が「1977 年に発表された」ことなど、暗号アルゴリズムの歴史的事実に触れました

しかし、暗号技術は突然登場したものではありません

「秘密の情報をどう守るか」という課題は数千年前から存在し、暗号技術はその課題に対する解決策として進化してきました

この補足資料では、古典暗号から現代の暗号標準に至るまでの歴史を辿ります

各時代の暗号がどのような課題を解決し、なぜ次の暗号が必要になったのかを知ることで、01-cryptography で学んだアルゴリズムの設計思想がより深く理解できるようになります

---

## [このページで学ぶこと](#what-you-will-learn) {#what-you-will-learn}

- 暗号技術の歴史的な発展の流れ
- 各時代の暗号が解決しようとした課題と、その限界
- 公開鍵暗号の革命がなぜ画期的だったのか
- 暗号が「破られる」とはどういうことか

---

## [目次](#table-of-contents) {#table-of-contents}

1. [年表](#timeline)
2. [古典暗号の時代](#classical-cryptography)
3. [機械式暗号の時代](#mechanical-cryptography)
4. [コンピュータ時代の幕開けと DES](#computer-era-and-des)
5. [公開鍵暗号の革命](#public-key-revolution)
6. [AES と公開コンペの意義](#aes-and-competition)
7. [暗号が「破られる」とはどういうことか](#what-means-cryptography-broken)
8. [用語集](#glossary)
9. [参考資料](#references)

---

## [年表](#timeline) {#timeline}

{: .labeled}
| 年代 | 出来事 |
| -------------- | ------------------------------------------------------------------ |
| 紀元前 50 年頃 | シーザー暗号（単純な文字のずらし） |
| 15 世紀 | ヴィジュネル暗号（多表式換字暗号） |
| 1918 年 | エニグマの特許取得（ドイツ） |
| 1939〜1945 年 | 第二次世界大戦でのエニグマ解読（Alan Turing ら） |
| 1949 年 | Claude Shannon が "Communication Theory of Secrecy Systems" を発表 |
| 1976 年 | DES が FIPS 46 として米国連邦標準に制定 |
| 1976 年 | Diffie と Hellman が "New Directions in Cryptography" を発表 |
| 1977 年 | Rivest、Shamir、Adleman が RSA を発表 |
| 1985 年 | Koblitz と Miller が独立に楕円曲線暗号を提案 |
| 1997 年 | DES が総当たり攻撃で解読される |
| 1997 年 | NIST が AES の公開コンペを開始 |
| 2001 年 | Rijndael が AES として選定（FIPS 197） |
| 2005 年 | Wang らが SHA-1 への衝突攻撃を発表 |
| 2017 年 | Google が SHA-1 の実用的な衝突を実証（SHAttered） |

---

## [古典暗号の時代](#classical-cryptography) {#classical-cryptography}

### [シーザー暗号](#caesar-cipher) {#caesar-cipher}

<strong>シーザー暗号</strong>は、ローマの将軍ユリウス・カエサル（Julius Caesar）が使ったとされる暗号で、文字を一定の数だけずらす方式です

たとえば「3 文字ずらす」というルールでは、A は D に、B は E に変換されます

```
平文：  HELLO
暗号文：KHOOR（各文字を 3 文字後ろにずらす）
```

シーザー暗号の鍵はずらす文字数（0〜25）であり、可能な鍵はわずか 26 通りです

26 通りすべてを試せば必ず解読できるため、暗号としての強度はほぼありません

しかし、シーザー暗号には暗号の基本構造が含まれています

<strong>鍵を使って平文を暗号文に変換し、同じ鍵で暗号文を平文に戻す</strong>という仕組みは、01-cryptography で学んだ対称暗号の原型です

### [ヴィジュネル暗号](#vigenere-cipher) {#vigenere-cipher}

<strong>ヴィジュネル暗号</strong>は、15 世紀にイタリアの暗号学者 Leon Battista Alberti の着想を基に発展し、16 世紀にフランスの外交官 Blaise de Vigenère が体系化した暗号です

シーザー暗号では全ての文字を同じ数だけずらしますが、ヴィジュネル暗号では<strong>文字ごとにずらす数を変える</strong>方式です

鍵として文字列（たとえば "KEY"）を使い、平文の各文字に対して鍵の文字を順番に適用します

```
平文：  H E L L O W O R L D
鍵：    K E Y K E Y K E Y K
暗号文：R I J V S U Y V J N
```

H を K の分（10）だけずらして R に、E を E の分（4）だけずらして I に、という具合です

ヴィジュネル暗号は数百年にわたって「解読不可能な暗号」と呼ばれていました

しかし、19 世紀に <strong>Friedrich Kasiski</strong> と <strong>Charles Babbage</strong> が独立に解読法を発見しました

鍵が繰り返されるため、暗号文中に同じパターンが出現し、その間隔から鍵の長さを推測できるのです

鍵の長さが判明すれば、問題はシーザー暗号と同じになります

### [古典暗号の限界](#classical-cryptography-limitations) {#classical-cryptography-limitations}

古典暗号に共通する弱点は、<strong>アルゴリズムの秘密に依存している</strong>ことです

暗号の仕組みそのものが知られると、鍵空間の小ささから解読が容易になります

19 世紀の暗号学者 <strong>Auguste Kerckhoffs</strong> は、この問題を指摘して以下の原則を提唱しました

> 暗号システムの安全性は、アルゴリズムの秘密ではなく、鍵の秘密だけに依存すべきである

この <strong>Kerckhoffs の原則</strong>は、現代の暗号設計の基盤です

01-cryptography で学んだ AES や RSA のアルゴリズムは完全に公開されていますが、鍵が秘密である限り安全です

---

## [機械式暗号の時代](#mechanical-cryptography) {#mechanical-cryptography}

### [エニグマ](#enigma) {#enigma}

20 世紀に入ると、暗号は手作業から機械へと移行しました

<strong>エニグマ</strong>は、1918 年にドイツで特許が取得された電気機械式の暗号装置です

タイプライターのような鍵盤を押すと、内部のローター（回転する円盤）が文字を複雑に置き換えます

エニグマの特徴は、1 文字を暗号化するたびにローターが回転し、<strong>同じ文字を続けて入力しても異なる暗号文が出力される</strong>ことです

ローターの数、初期位置、配線の設定を組み合わせると、可能な鍵の数は 10 の 23 乗を超えます

この膨大な鍵空間により、第二次世界大戦でドイツ軍はエニグマを「解読不可能」と信じていました

### [エニグマの解読](#enigma-decryption) {#enigma-decryption}

エニグマの解読は、ポーランドの暗号学者 <strong>Marian Rejewski</strong> らの研究に始まり、イギリスの <strong>Alan Turing</strong> らがブレッチリー・パークで完成させました

Turing らは、エニグマの設計上の弱点を利用しました

たとえば、エニグマでは<strong>ある文字が自分自身に暗号化されることがない</strong>という性質がありました

この性質を利用し、暗号文中に特定のパターン（天気予報の定型文など）が含まれていると推測して、可能な設定を絞り込みました

エニグマの解読は、暗号の歴史において重要な教訓を残しました

<strong>鍵空間が大きいだけでは安全ではない</strong>ということです

アルゴリズムに構造的な弱点があれば、鍵空間を全探索しなくても解読できます

---

## [コンピュータ時代の幕開けと DES](#computer-era-and-des) {#computer-era-and-des}

### [Shannon の情報理論](#shannon-information-theory) {#shannon-information-theory}

1949 年、<strong>Claude Shannon</strong> が "Communication Theory of Secrecy Systems" を発表しました

この論文は、暗号の安全性を数学的に分析する枠組みを初めて提供したものです

Shannon は、暗号文から平文についての情報が一切得られない状態を<strong>完全秘匿（perfect secrecy）</strong>と定義しました

そして、完全秘匿を達成するには<strong>鍵の長さが平文の長さ以上</strong>でなければならないことを証明しました

これは<strong>ワンタイムパッド</strong>（平文と同じ長さのランダムな鍵を一度だけ使う暗号）が唯一の完全秘匿暗号であることを意味します

しかし、平文と同じ長さの鍵を安全に配送・管理するのは現実的ではありません

そのため、現代の暗号は完全秘匿ではなく、<strong>計算量的安全性</strong>を目標としています

計算量的安全性とは、「理論上は解読可能だが、現実的な計算資源では解読に膨大な時間がかかる」という安全性です

01-cryptography で学んだ AES や RSA は、この計算量的安全性に基づいています

### [DES の誕生](#des-birth) {#des-birth}

1970 年代、コンピュータの普及に伴い、デジタルデータの暗号化が必要になりました

1973 年、米国国立標準技術研究所（NIST、当時は NBS）が暗号アルゴリズムの公募を行いました

IBM が提出した <strong>Lucifer</strong> をベースに、NSA（国家安全保障局）の助言を受けて修正されたアルゴリズムが、1976 年に <strong>DES</strong>（Data Encryption Standard）として米国連邦標準に制定されました

DES は<strong>鍵長 56 ビット</strong>のブロック暗号で、01-cryptography で学んだ対称暗号の一種です

DES の標準化には 2 つの歴史的意義がありました

1 つ目は、<strong>暗号が初めて公開標準</strong>になったことです

それまで暗号は軍や情報機関の秘密でしたが、DES により民間でも同じ暗号を使えるようになりました

2 つ目は、<strong>NSA の関与に対する懸念</strong>が暗号の透明性についての議論を生んだことです

IBM の Lucifer は元々 128 ビットの鍵長でしたが、DES では 56 ビットに短縮されました

NSA がバックドアを仕込んだのではないかという疑念が広がりましたが、後に NSA の修正は差分解読法への耐性を強化するものだったことが判明しています

### [DES の終焉](#des-end) {#des-end}

コンピュータの処理能力が向上するにつれ、56 ビットの鍵長は不十分になりました

1997 年、<strong>RSA Security 社</strong>が主催した DES 解読コンテストで、DES は<strong>総当たり攻撃</strong>（考えられる全ての鍵を試す方法）で解読されました

2 の 56 乗（約 7200 兆）通りの鍵を全て探索するのに、分散コンピューティングで数ヶ月、1999 年には専用ハードウェア（EFF の "Deep Crack"）で<strong>22 時間 15 分</strong>で解読されました

DES の終焉は、<strong>鍵長は暗号の寿命を決定する</strong>という教訓を残しました

コンピュータの性能は年々向上するため、一定の鍵長は時間とともに安全性を失います

---

## [公開鍵暗号の革命](#public-key-revolution) {#public-key-revolution}

### [Diffie-Hellman 論文](#diffie-hellman-paper) {#diffie-hellman-paper}

1976 年、<strong>Whitfield Diffie</strong> と <strong>Martin Hellman</strong> が "New Directions in Cryptography" を発表しました

この論文は、暗号の歴史における最も重要な転換点の 1 つです

DES をはじめとするそれまでの暗号は全て<strong>対称暗号</strong>であり、送信者と受信者が事前に同じ鍵を共有する必要がありました

しかし、インターネットのように不特定多数が通信する環境では、全ての通信相手と事前に鍵を共有するのは現実的ではありません

Diffie と Hellman は、<strong>公開の通信路を使って安全に共有秘密を作り出す</strong>方法を提案しました

これが 01-cryptography で学んだ <strong>Diffie-Hellman 鍵交換</strong>です

また、この論文では「暗号化と復号に異なる鍵を使う暗号」の概念も提唱されました

暗号化の鍵を公開しても復号の鍵を知られなければ安全である、という<strong>公開鍵暗号</strong>のアイデアです

### [RSA の登場](#rsa-emergence) {#rsa-emergence}

Diffie-Hellman 論文は公開鍵暗号の概念を提唱しましたが、具体的な暗号化アルゴリズムは示していませんでした

1977 年、MIT の <strong>Ron Rivest</strong>、<strong>Adi Shamir</strong>、<strong>Leonard Adleman</strong> が、公開鍵暗号の最初の実用的な実装を発表しました

3 人の頭文字をとって <strong>RSA</strong> と名付けられたこのアルゴリズムは、01-cryptography で学んだとおり、<strong>大きな数の素因数分解の困難さ</strong>に安全性を依存しています

RSA の登場により、<strong>鍵配送問題</strong>が解決されました

見知らぬ相手に暗号化された通信を送りたいとき、相手の公開鍵を入手するだけでよくなりました

これがインターネット時代の暗号化通信の基盤となりました

### [楕円曲線暗号](#elliptic-curve-cryptography) {#elliptic-curve-cryptography}

1985 年、<strong>Neal Koblitz</strong> と <strong>Victor Miller</strong> が独立に<strong>楕円曲線暗号（ECC）</strong>を提案しました

01-cryptography で学んだとおり、楕円曲線暗号は<strong>楕円曲線上の離散対数問題の困難さ</strong>に安全性を依存しています

楕円曲線暗号の革新は、RSA と同等の安全性を<strong>はるかに短い鍵長</strong>で実現できることです

{: .labeled}
| 安全性レベル | RSA の鍵長 | 楕円曲線暗号の鍵長 |
| ------------ | ------------ | ------------------ |
| 128 ビット | 3072 ビット | 256 ビット |
| 256 ビット | 15360 ビット | 512 ビット |

鍵が短いということは、計算量が少なく、通信データのサイズも小さくなります

このため、02-tls で学んだ TLS 1.3 では、鍵交換に <strong>ECDHE</strong>（楕円曲線 Diffie-Hellman Ephemeral）が標準的に使われています

---

## [AES と公開コンペの意義](#aes-and-competition) {#aes-and-competition}

### [DES の後継を求めて](#des-successor) {#des-successor}

DES が 56 ビットの鍵長で安全性を失ったことを受け、NIST は 1997 年に <strong>AES</strong>（Advanced Encryption Standard）の公開コンペを開始しました

AES コンペは、DES の標準化で問題となった<strong>透明性</strong>を徹底しました

{: .labeled}
| DES の標準化（1970 年代） | AES の公開コンペ（1997〜2001 年） |
| ------------------------- | ---------------------------------- |
| IBM が提出し NSA が修正 | 世界中の研究者が応募 |
| 修正理由が非公開 | 評価基準と評価過程が公開 |
| NSA のバックドア疑惑 | 暗号学者コミュニティによる公開検証 |

### [コンペの過程](#competition-process) {#competition-process}

AES コンペには世界中から 15 の候補が提出されました

評価は以下の基準で行われました

- <strong>安全性</strong>：既知の攻撃手法に対する耐性
- <strong>効率性</strong>：ソフトウェアとハードウェアでの実装性能
- <strong>柔軟性</strong>：異なる鍵長（128、192、256 ビット）への対応

2000 年に 5 候補（Rijndael、Serpent、Twofish、RC6、MARS）に絞られ、2001 年にベルギーの暗号学者 <strong>Joan Daemen</strong> と <strong>Vincent Rijmen</strong> が設計した <strong>Rijndael</strong> が AES として選定されました

### [公開コンペの意義](#public-competition-significance) {#public-competition-significance}

AES コンペの最大の意義は、暗号の<strong>標準化プロセスの透明性</strong>を確立したことです

暗号アルゴリズムは秘密にすることで安全になるのではなく、<strong>公開して多くの専門家に検証されることで安全性が確認される</strong>という考え方です

これは Kerckhoffs の原則の実践であり、以降の暗号標準化（SHA-3 の選定など）でも同じアプローチが採用されています

01-cryptography で学んだ AES が広く信頼されているのは、このアルゴリズムが公開の場で徹底的に検証された結果、重大な弱点が発見されなかったためです

---

## [暗号が「破られる」とはどういうことか](#what-means-cryptography-broken) {#what-means-cryptography-broken}

暗号の歴史を振り返ると、「暗号が破られた」という表現が繰り返し登場します

しかし、「破られる」にはいくつかの段階があります

### [破られ方の分類](#breaking-classification) {#breaking-classification}

{: .labeled}
| 段階 | 説明 | 影響 |
| ------------------ | ---------------------------------------------- | ------------------------------------------------ |
| 理論的な弱点の発見 | 総当たりより効率的な攻撃方法が見つかる | 即座に危険ではないが、安全性の余裕が減る |
| 実用的な攻撃の成功 | 現実的な計算資源で解読が可能になる | 移行が急務になる |
| 鍵長の陳腐化 | コンピュータの性能向上で総当たりが現実的になる | 鍵長の拡大または新しいアルゴリズムへの移行が必要 |

### [SHA-1 の事例](#sha1-case) {#sha1-case}

ハッシュ関数 SHA-1 の「破られ方」は、この段階的な過程をよく示しています

01-cryptography で学んだとおり、ハッシュ関数の安全性の 1 つに<strong>衝突耐性</strong>があります

衝突耐性とは、同じハッシュ値を持つ 2 つの異なる入力を見つけるのが困難であるという性質です

SHA-1 は 160 ビットのハッシュ値を出力するため、理論上は 2 の 80 乗の計算で衝突を見つけられます

2005 年、<strong>Xiaoyun Wang</strong> らは SHA-1 の衝突を 2 の 69 乗程度の計算で見つけられることを示しました

これは理論的な弱点の発見であり、2 の 80 乗と比べて 2000 倍以上効率的ですが、当時の計算資源ではまだ実用的ではありませんでした

2017 年、Google は <strong>SHAttered</strong> プロジェクトで、実際に異なる内容の 2 つの PDF ファイルが同じ SHA-1 ハッシュ値を持つことを実証しました

これは実用的な攻撃の成功であり、SHA-1 は衝突耐性が必要な用途では安全ではなくなりました

SHA-1 から SHA-256（SHA-2 ファミリー）への移行は段階的に進められ、[03-certificate](../../03-certificate/) で学んだ証明書の署名アルゴリズムも SHA-256 が標準となっています

### [暗号の寿命](#cryptography-lifetime) {#cryptography-lifetime}

暗号の歴史は、「破られない暗号」が存在しないことを繰り返し示しています

全ての暗号には寿命があり、その寿命はコンピュータの性能向上と暗号解読研究の進展によって決まります

このため、暗号の設計では<strong>安全性の余裕</strong>（security margin）が重要です

たとえば、AES-256 は 256 ビットの鍵を使いますが、128 ビットの安全性で十分と考えられる場面でも 256 ビットを選ぶことで、将来の攻撃手法の進歩に対する余裕を持たせることができます

また、アルゴリズムの<strong>移行計画</strong>も暗号運用の重要な要素です

DES から AES への移行、SHA-1 から SHA-256 への移行のように、あるアルゴリズムが弱くなる前に次のアルゴリズムに移行する準備を進めておく必要があります

---

## [用語集](#glossary) {#glossary}

{: .labeled}
| 用語 | 説明 |
| ----------------- | ------------------------------------------------------------------------------------------ |
| シーザー暗号 | 文字を一定数ずらす古典的な暗号方式 |
| ヴィジュネル暗号 | 鍵文字列を使って文字ごとにずらす数を変える多表式換字暗号 |
| Kerckhoffs の原則 | 暗号の安全性はアルゴリズムの秘密ではなく鍵の秘密だけに依存すべきという原則 |
| エニグマ | 第二次世界大戦でドイツ軍が使用した電気機械式暗号装置 |
| 完全秘匿 | 暗号文から平文についての情報が一切得られない状態（Shannon が定義） |
| 計算量的安全性 | 現実的な計算資源では解読に膨大な時間がかかるという安全性の概念 |
| DES | Data Encryption Standard の略で、1976 年に米国連邦標準に制定された鍵長 56 ビットの対称暗号 |
| AES | Advanced Encryption Standard の略で、2001 年に公開コンペで選定された対称暗号 |
| 安全性の余裕 | 既知の最良の攻撃手法と暗号の設計強度との差 |

---

## [参考資料](#references) {#references}

<strong>Shannon の論文</strong>

- [Communication Theory of Secrecy Systems](https://ieeexplore.ieee.org/document/6769090){:target="\_blank"}
  - Claude Shannon による暗号の情報理論的分析（1949 年）

<strong>Diffie-Hellman 論文</strong>

- [New Directions in Cryptography](https://ieeexplore.ieee.org/document/1055638){:target="\_blank"}
  - Whitfield Diffie と Martin Hellman による公開鍵暗号の概念の提唱（1976 年）

<strong>NIST 暗号標準</strong>

- [FIPS 197 - Advanced Encryption Standard (AES)](https://csrc.nist.gov/pubs/fips/197/final){:target="\_blank"}
  - AES の仕様

- [FIPS 46-3 - Data Encryption Standard (DES)](https://csrc.nist.gov/pubs/fips/46-3/final){:target="\_blank"}
  - DES の仕様（1999 年に廃止）

<strong>SHA-1 衝突</strong>

- [SHAttered](https://shattered.io/){:target="\_blank"}
  - Google による SHA-1 の実用的な衝突実証（2017 年）

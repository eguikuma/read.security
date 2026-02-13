---
layout: default
title: OAuth 2.0 と OpenID Connect の全体像
---

# [appendix：OAuth 2.0 と OpenID Connect の全体像](#oauth-openid-connect) {#oauth-openid-connect}

## [はじめに](#introduction) {#introduction}

[04-authentication](../../04-authentication/) では、OAuth 2.0 の<strong>認可コードフロー</strong>を詳しく学びました

サードパーティアプリケーションがユーザーの代わりにリソースにアクセスするための「委譲された権限」の仕組みです

04-authentication では、認可コードフローと PKCE を中心に解説しましたが、OAuth 2.0 にはほかにもいくつかのフローがあります

また、OAuth 2.0 は<strong>認可</strong>のためのプロトコルであり、<strong>認証</strong>の仕組みは含まれていません

「OAuth 2.0 でログインする」という表現は広く使われていますが、実際に認証を提供しているのは OAuth 2.0 を拡張した <strong>OpenID Connect</strong> です

04-authentication の用語集では「OAuth 2.0 を拡張して認証機能を追加したプロトコル」と一行で紹介しましたが、具体的な仕組みは説明しませんでした

さらに、[07-supply-chain](../../07-supply-chain/) で学んだ Sigstore の Fulcio は、<strong>OIDC 認証</strong>を使って署名者の身元を確認しています

この補足資料では、OAuth 2.0 の全体像と OpenID Connect の仕組みを学びます

---

## [このページで学ぶこと](#what-you-will-learn) {#what-you-will-learn}

- OAuth 2.0 の 4 つのグラントタイプと、それぞれの用途
- OAuth 2.0 が「認証」ではない理由
- OpenID Connect が OAuth 2.0 に何を追加するか
- ID トークンの構造と検証方法
- Sigstore における OIDC の役割

---

## [目次](#table-of-contents) {#table-of-contents}

1. [OAuth 2.0 のグラントタイプ](#oauth-grant-types)
2. [OAuth 2.0 は認証ではない](#oauth-is-not-authentication)
3. [OpenID Connect とは](#what-is-openid-connect)
4. [ID トークン](#id-token)
5. [認可コードフロー + OpenID Connect](#authorization-code-flow-with-oidc)
6. [Sigstore における OIDC](#sigstore-oidc)
7. [まとめ](#summary)
8. [用語集](#glossary)
9. [参考資料](#references)

---

## [OAuth 2.0 のグラントタイプ](#oauth-grant-types) {#oauth-grant-types}

OAuth 2.0（RFC 6749）は、アクセストークンを取得するための方法として<strong>4 つのグラントタイプ</strong>を定義しています

04-authentication で詳しく学んだ認可コードフローは、このうちの 1 つです

{: .labeled}
| グラントタイプ | 用途 | 安全性 |
| ---------------------------------------------- | --------------------------------------------------- | -------------- |
| 認可コードフロー | Web アプリケーション、モバイルアプリ（PKCE と併用） | 高い |
| インプリシットフロー | かつてのブラウザ内 JavaScript アプリ向け | 低い（非推奨） |
| リソースオーナーパスワードクレデンシャルフロー | ユーザーのパスワードを直接受け取る | 低い（非推奨） |
| クライアントクレデンシャルフロー | サーバー間通信（ユーザーが介在しない） | 用途による |

### [認可コードフロー](#authorization-code-flow) {#authorization-code-flow}

04-authentication で学んだとおり、<strong>認可コードフロー</strong>は最も安全なグラントタイプです

ユーザーが認可サーバーで認証し、認可コードがクライアントに返されます

クライアントはこの認可コードをアクセストークンに交換します

アクセストークンがブラウザに直接露出しないため、トークンの漏洩リスクが低くなります

### [インプリシットフロー（非推奨）](#implicit-flow) {#implicit-flow}

<strong>インプリシットフロー</strong>は、認可コードを経由せずに、認可サーバーからアクセストークンを直接返すフローです

認可コードの交換ステップを省略するため、かつてはブラウザ内の JavaScript アプリケーション向けに推奨されていました

しかし、アクセストークンがブラウザの URL フラグメントに露出するため、トークンの漏洩リスクが高く、PKCE 付きの認可コードフローが推奨されるようになりました

OAuth 2.1（策定中）では、インプリシットフローは仕様から削除される予定です

### [リソースオーナーパスワードクレデンシャルフロー（非推奨）](#resource-owner-password-flow) {#resource-owner-password-flow}

このフローでは、ユーザーのパスワードをクライアントアプリケーションに直接入力します

クライアントがパスワードを受け取り、認可サーバーに送信してアクセストークンを取得します

04-authentication で学んだ OAuth 2.0 の本来の目的（パスワードを第三者に渡さない）に反するため、非推奨です

### [クライアントクレデンシャルフロー](#client-credentials-flow) {#client-credentials-flow}

<strong>クライアントクレデンシャルフロー</strong>は、ユーザーが介在しないサーバー間通信で使われます

クライアント自身の認証情報（client_id と client_secret）を使ってアクセストークンを取得します

たとえば、バックエンドサービスが別の API にアクセスする場合などに使われます

---

## [OAuth 2.0 は認証ではない](#oauth-is-not-authentication) {#oauth-is-not-authentication}

### [認可と認証の違い](#authorization-and-authentication-difference) {#authorization-and-authentication-difference}

04-authentication で学んだとおり、<strong>認証</strong>と<strong>認可</strong>は異なる概念です

- <strong>認証</strong>：「あなたは誰か」を確認する
- <strong>認可</strong>：「あなたは何をしてよいか」を決定する

OAuth 2.0 は<strong>認可</strong>のためのプロトコルです

アクセストークンは「このクライアントは、このユーザーの代わりに、このスコープの操作を許可されている」ことを示しますが、「このユーザーが誰であるか」については何も定義していません

### [OAuth 2.0 を認証に使う問題](#oauth-as-authentication-problem) {#oauth-as-authentication-problem}

OAuth 2.0 のアクセストークンを使って「ユーザーの認証」を行おうとする実装があります

たとえば、アクセストークンを使ってユーザー情報 API を呼び出し、返ってきたユーザー情報で「ログイン済み」とみなすような実装です

しかし、この方法にはセキュリティ上の問題があります

{: .labeled}
| 問題 | 説明 |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------- |
| トークンの対象者が不明 | アクセストークンには「誰が認証したか」「どのクライアント向けに発行されたか」の情報が標準化されていない |
| トークンの置き換え攻撃 | 悪意のあるクライアントが、別のクライアント向けに発行されたアクセストークンを使ってユーザーになりすませる可能性がある |
| 認証のタイミングが不明 | アクセストークンは「いつユーザーが認証したか」を保証しない |

これらの問題を解決するために、OAuth 2.0 の上に認証の仕組みを追加したのが <strong>OpenID Connect</strong> です

---

## [OpenID Connect とは](#what-is-openid-connect) {#what-is-openid-connect}

<strong>OpenID Connect</strong>（OIDC）は、OAuth 2.0 を拡張して<strong>認証</strong>の機能を追加したプロトコルです

OpenID Connect は OAuth 2.0 の仕組みを<strong>そのまま利用</strong>しつつ、以下を追加しています

{: .labeled}
| 追加要素 | 説明 |
| ----------------------- | -------------------------------------------------- |
| ID トークン | ユーザーの認証情報を含む JWT |
| UserInfo エンドポイント | ユーザーの属性情報を取得するための標準 API |
| 標準スコープ（openid） | 認証を要求することを示すスコープ |
| 標準クレーム | ユーザー情報の表現方法の標準化（name、email など） |

OAuth 2.0 が「このクライアントはリソースにアクセスしてよい」を扱うのに対し、OpenID Connect は「このユーザーは確かに認証された」を扱います

両者の関係を整理すると以下のようになります

|                  | OAuth 2.0                            | OpenID Connect                                  |
| ---------------- | ------------------------------------ | ----------------------------------------------- |
| 目的             | 認可（リソースへのアクセス許可）     | 認証（ユーザーの身元確認）                      |
| 発行するトークン | アクセストークン                     | アクセストークン + <strong>ID トークン</strong> |
| 回答する問い     | 「このクライアントは何をしてよいか」 | 「このユーザーは誰か」                          |

---

## [ID トークン](#id-token) {#id-token}

### [ID トークンの構造](#id-token-structure) {#id-token-structure}

<strong>ID トークン</strong>は、04-authentication で学んだ <strong>JWT</strong>（JSON Web Token）形式で表現されます

ID トークンには、ユーザーの認証に関する情報が<strong>クレーム（claims）</strong>として含まれています

{: .labeled}
| クレーム | 説明 |
| ----------------- | ------------------------------------------------------------ |
| iss（Issuer） | ID トークンを発行した認可サーバー（OpenID Provider）の識別子 |
| sub（Subject） | ユーザーの一意識別子 |
| aud（Audience） | このトークンの対象となるクライアントの client_id |
| exp（Expiration） | トークンの有効期限 |
| iat（Issued At） | トークンの発行時刻 |
| nonce | リプレイ攻撃を防ぐためのランダムな値 |
| auth_time | ユーザーが認証を行った時刻 |

### [ID トークンの検証](#id-token-validation) {#id-token-validation}

クライアントは ID トークンを受け取った後、以下の検証を行います

1. <strong>署名の検証</strong>：認可サーバーの公開鍵で JWT の署名を検証し、改ざんされていないことを確認する
2. <strong>iss の確認</strong>：期待する認可サーバーから発行されたことを確認する
3. <strong>aud の確認</strong>：自分の client_id が aud に含まれていることを確認する（他のクライアント向けのトークンでないことの検証）
4. <strong>exp の確認</strong>：トークンが有効期限内であることを確認する
5. <strong>nonce の確認</strong>：認可リクエスト時に送った nonce と一致することを確認する（リプレイ攻撃の防止）

aud の検証は特に重要です

これにより、OAuth 2.0 単体で認証を行う場合に問題となった「トークンの置き換え攻撃」を防ぐことができます

悪意のあるクライアント向けに発行された ID トークンには、そのクライアントの client_id が aud に設定されているため、別のクライアントでは検証に失敗します

---

## [認可コードフロー + OpenID Connect](#authorization-code-flow-with-oidc) {#authorization-code-flow-with-oidc}

04-authentication で学んだ認可コードフローに OpenID Connect を組み合わせると、認可と認証を同時に行えます

### [フローの概要](#flow-overview) {#flow-overview}

<strong>1. 認可リクエスト</strong>

クライアントがユーザーを認可サーバーにリダイレクトする際、スコープに <strong>openid</strong> を含めます

openid スコープは「認証を要求する」ことを意味し、認可サーバーに ID トークンの発行を求めます

<strong>2. ユーザー認証と認可</strong>

ユーザーが認可サーバーでログインし、クライアントへのアクセス許可を与えます

<strong>3. 認可コードの発行</strong>

04-authentication と同様に、認可コードがクライアントにリダイレクトで返されます

<strong>4. トークンの交換</strong>

クライアントが認可コードを認可サーバーのトークンエンドポイントに送信すると、<strong>アクセストークンと ID トークンの両方</strong>が返されます

OAuth 2.0 単体ではアクセストークンのみが返されますが、OpenID Connect ではスコープに openid が含まれている場合に ID トークンも発行されます

<strong>5. ID トークンの検証</strong>

クライアントは ID トークンの署名、iss、aud、exp、nonce を検証し、ユーザーの認証情報を確認します

---

## [Sigstore における OIDC](#sigstore-oidc) {#sigstore-oidc}

07-supply-chain で学んだ Sigstore の <strong>Fulcio</strong> は、OpenID Connect を使ってソフトウェア署名者の身元を確認しています

### [Fulcio の仕組み](#fulcio-mechanism) {#fulcio-mechanism}

Fulcio は<strong>短期署名証明書</strong>を発行する認証局です

07-supply-chain で学んだとおり、従来のソフトウェア署名では長期的な秘密鍵の管理が課題でした

Fulcio は OIDC を使うことで、鍵の管理を不要にしています

1. 開発者が GitHub や Google などの <strong>OpenID Provider</strong> で認証する
2. OpenID Provider が <strong>ID トークン</strong>を発行する
3. 開発者が ID トークンを Fulcio に提出する
4. Fulcio は ID トークンの署名を検証し、開発者の身元（メールアドレスなど）を確認する
5. Fulcio は開発者の公開鍵に対して<strong>短期証明書</strong>（有効期間約 10 分）を発行する

### [OIDC が Sigstore にもたらすもの](#oidc-for-sigstore) {#oidc-for-sigstore}

{: .labeled}
| 従来のソフトウェア署名 | Sigstore + OIDC |
| -------------------------- | -------------------------------------------------------- |
| 長期的な秘密鍵の管理が必要 | 秘密鍵は使い捨て（短期証明書の有効期間中のみ） |
| 鍵の配布と信頼の確立が必要 | 既存の OIDC プロバイダー（GitHub、Google）の信頼を再利用 |
| 鍵の漏洩が長期的なリスク | 証明書の有効期間が短いため、漏洩のリスクが限定的 |

Sigstore が OIDC を採用したことで、[03-certificate](../../03-certificate/) で学んだ PKI の概念（証明書、認証局、信頼の連鎖）と、04-authentication で学んだ認証の概念が、ソフトウェアサプライチェーンの文脈で結びつきます

---

## [まとめ](#summary) {#summary}

{: .labeled}
| ポイント | 説明 |
| -------------------------------------------------- | ---------------------------------------------------------------------------- |
| OAuth 2.0 には 4 つのグラントタイプがある | 認可コードフローが最も安全で、インプリシットフローとパスワードフローは非推奨 |
| OAuth 2.0 は認可のプロトコルであり認証ではない | アクセストークンは「何をしてよいか」を示すが「誰か」は示さない |
| OpenID Connect は OAuth 2.0 に認証を追加する | ID トークン（JWT）により「このユーザーは確かに認証された」を証明 |
| ID トークンの aud 検証がトークン置き換え攻撃を防ぐ | トークンが特定のクライアント向けに発行されたことを保証 |
| Sigstore は OIDC で署名者の身元を確認する | 既存の認証基盤を再利用し、長期的な鍵管理を不要にする |

---

## [用語集](#glossary) {#glossary}

{: .labeled}
| 用語 | 説明 |
| -------------------------------- | -------------------------------------------------------------------- |
| グラントタイプ | OAuth 2.0 でアクセストークンを取得するためのフローの種類 |
| インプリシットフロー | 認可コードを経由せずにアクセストークンを直接返すフロー（非推奨） |
| クライアントクレデンシャルフロー | ユーザーが介在しないサーバー間通信で使われるフロー |
| OpenID Connect | OAuth 2.0 を拡張して認証機能を追加したプロトコル |
| ID トークン | ユーザーの認証情報を含む JWT |
| OpenID Provider | OpenID Connect における認証を行うサーバー（認可サーバー + 認証機能） |
| クレーム | JWT に含まれる属性情報（iss、sub、aud、exp など） |
| UserInfo エンドポイント | ユーザーの属性情報を取得するための標準 API |

---

## [参考資料](#references) {#references}

<strong>OAuth 2.0</strong>

- [RFC 6749 - The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749){:target="\_blank"}
  - OAuth 2.0 の仕様

<strong>OpenID Connect</strong>

- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html){:target="\_blank"}
  - OpenID Connect の仕様

<strong>Sigstore</strong>

- [Fulcio - Free Code Signing](https://docs.sigstore.dev/certificate_authority/overview/){:target="\_blank"}
  - Sigstore の認証局 Fulcio の概要

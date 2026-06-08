---
tags:
  - API
  - REST
  - OpenAPI
  - HTTP
created: 2026-05-25
updated: 2026-05-25
---

# REST API 設計リファレンス

## 目次

- [概要](#概要)
- [REST API の基本原則](#rest-api-の基本原則)
  - [リソース設計](#リソース設計)
  - [HTTP メソッド（CRUD）](#http-メソッドcrud)
  - [ステータスコード](#ステータスコード)
  - [URL 設計](#url-設計)
- [リクエスト・レスポンス](#リクエストレスポンス)
  - [リクエスト](#リクエスト)
  - [メディアタイプ](#メディアタイプ)
  - [レスポンス形式](#レスポンス形式)
  - [エラーレスポンス](#エラーレスポンス)
- [クエリパラメータ](#クエリパラメータ)
  - [フィルタリング・検索](#フィルタリング検索)
  - [ページネーション](#ページネーション)
  - [ソート](#ソート)
- [バージョニング](#バージョニング)
- [認証・認可](#認証認可)
  - [Bearer トークン](#bearer-トークン)
  - [API キー](#api-キー)
- [キャッシュ・条件付きリクエスト](#キャッシュ条件付きリクエスト)
- [レート制限](#レート制限)
- [非同期処理](#非同期処理)
- [OpenAPI（Swagger）](#openapiswagger)
  - [基本構造](#基本構造)
  - [パス定義](#パス定義)
  - [スキーマ定義](#スキーマ定義)
  - [認証の定義](#認証の定義)
  - [運用](#運用)
- [設計のベストプラクティス](#設計のベストプラクティス)
  - [命名規則](#命名規則)
  - [互換性](#互換性)
  - [安全性と冪等性](#安全性と冪等性)
  - [リトライと Idempotency-Key](#リトライと-idempotency-key)
  - [PATCH の形式](#patch-の形式)
  - [HATEOAS](#hateoas)
- [用語集](#用語集)
- [リンク集](#リンク集)

---

## 概要

REST API は HTTP プロトコルを用いてリソースを操作する設計スタイル。  
OpenAPI（旧 Swagger）はその仕様を YAML/JSON で記述するための標準フォーマット。

---

## REST API の基本原則

### リソース設計

- **名詞を使う**（動詞はメソッドで表現）
- **複数形**を使う
- リソースの階層はパスで表現する

```
# Good
GET /users
GET /users/{id}
GET /users/{id}/posts

# Bad
GET /getUser
POST /createPost
```

### HTTP メソッド（CRUD）

| メソッド | 操作       | 冪等 | 例                       |
|----------|------------|------|--------------------------|
| GET      | 取得       | ○    | `GET /users`             |
| POST     | 作成       | ✗    | `POST /users`            |
| PUT      | 完全置換   | ○    | `PUT /users/{id}`        |
| PATCH    | 部分更新   | △    | `PATCH /users/{id}`      |
| DELETE   | 削除       | ○    | `DELETE /users/{id}`     |

> PUT はリソース全体を置き換える。PATCH は指定フィールドのみ更新する。

### ステータスコード

**2xx — 成功**

| コード | 意味                                       |
|--------|--------------------------------------------|
| 200    | OK — 取得・更新成功                        |
| 201    | Created — 作成成功（`Location` ヘッダを付けることが多い）|
| 204    | No Content — 削除成功など、ボディなし      |

**4xx — クライアントエラー**

| コード | 意味                                         |
|--------|----------------------------------------------|
| 400    | Bad Request — リクエスト形式が不正           |
| 401    | Unauthorized — 未認証、または認証に失敗      |
| 403    | Forbidden — 認証済みだがアクセス権なし       |
| 404    | Not Found — リソースが存在しない             |
| 409    | Conflict — 競合（重複作成など）              |
| 422    | Unprocessable Entity — バリデーションエラー  |
| 429    | Too Many Requests — レート制限               |

**5xx — サーバーエラー**

| コード | 意味                                         |
|--------|----------------------------------------------|
| 500    | Internal Server Error — 予期しないエラー     |
| 503    | Service Unavailable — メンテナンス・過負荷   |

### URL 設計

```
# コレクション
GET    /articles          # 一覧取得
POST   /articles          # 新規作成

# 単一リソース
GET    /articles/{id}     # 取得
PUT    /articles/{id}     # 完全置換
PATCH  /articles/{id}     # 部分更新
DELETE /articles/{id}     # 削除

# ネストリソース
GET    /articles/{id}/comments        # 記事のコメント一覧
POST   /articles/{id}/comments        # コメント追加
DELETE /articles/{id}/comments/{cid}  # コメント削除

# アクション（動詞が必要なとき）
POST   /articles/{id}/publish   # 公開する
POST   /orders/{id}/cancel      # キャンセルする
```

---

## リクエスト・レスポンス

### リクエスト

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer <token>

{
  "name": "Ken Ishikawa",
  "email": "ken@example.com"
}
```

### メディアタイプ

リクエストボディの形式は `Content-Type`、期待するレスポンス形式は `Accept` で表現する。

```http
Content-Type: application/json
Accept: application/json
```

エラーレスポンスに Problem Details を使う場合は `application/problem+json` を返す。

```http
Content-Type: application/problem+json
```

### レスポンス形式

**単一リソース**

```json
{
  "id": "u_123",
  "name": "Ken Ishikawa",
  "email": "ken@example.com",
  "createdAt": "2026-05-25T10:00:00Z"
}
```

**コレクション（オフセットページネーション）**

```json
{
  "data": [
    { "id": "u_123", "name": "Ken Ishikawa" },
    { "id": "u_124", "name": "Taro Yamada" }
  ],
  "pagination": {
    "total": 100,
    "page": 1,
    "perPage": 20
  }
}
```

**コレクション（カーソルページネーション）**

```json
{
  "data": [
    { "id": "u_123", "name": "Ken Ishikawa" },
    { "id": "u_124", "name": "Taro Yamada" }
  ],
  "pagination": {
    "limit": 20,
    "nextCursor": "eyJjcmVhdGVkQXQiOiIyMDI2LTA1LTI1VDEwOjAwOjAwWiIsImlkIjoidV8xMjQifQ"
  }
}
```

### エラーレスポンス

RFC 9457（Problem Details for HTTP APIs）形式が標準的。RFC 7807 の後継。

```json
{
  "type": "https://example.com/errors/validation",
  "title": "Validation Error",
  "status": 422,
  "detail": "The email field is required.",
  "errors": [
    { "field": "email", "message": "required" }
  ]
}
```

---

## クエリパラメータ

### フィルタリング・検索

```
GET /articles?status=published
GET /articles?tag=go&tag=api
GET /articles?q=openapi
```

### ページネーション

**オフセット方式**（シンプルだが大量データで遅い）

```
GET /articles?page=2&perPage=20
GET /articles?offset=40&limit=20
```

**カーソル方式**（大量データに強い）

```
GET /articles?cursor=eyJjcmVhdGVkQXQiOiIyMDI2LTA1LTI1VDEwOjAwOjAwWiIsImlkIjoiYV8xMjQifQ&limit=20
```

カーソルはクライアントが中身に依存しない opaque な値として扱う。実装では `createdAt` や `id` など、安定したソートキーを JSON 化して Base64URL エンコードすることが多い。

### ソート

```
GET /articles?sort=createdAt             # 昇順
GET /articles?sort=-createdAt            # 降順（- プレフィックス）
GET /articles?sort=-createdAt,title      # 複数キー
GET /articles?sortBy=createdAt&order=desc
```

---

## バージョニング

| 方式             | 例                                        | 特徴                            |
|------------------|-------------------------------------------|---------------------------------|
| URL パス         | `/v1/users`                               | 最も一般的、キャッシュしやすい  |
| クエリパラメータ | `/users?version=1`                        | URL がシンプル、見落としやすい  |
| ヘッダー         | `Accept: application/vnd.api+json;v=1`    | クリーンな URL、扱いが複雑      |

URL パスが最もわかりやすく推奨されることが多い。

---

## 認証・認可

### Bearer トークン

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Bearer トークンには JWT（JSON Web Token）や opaque token が使われる。

`Authorization` ヘッダーは API クライアントから扱いやすい一方、ブラウザアプリでは XSS 対策が重要。Cookie を使う場合は `HttpOnly`、`Secure`、`SameSite`、CSRF 対策を考慮する。

### API キー

```http
X-API-Key: your-api-key
# または
Authorization: ApiKey your-api-key
```

サーバー間通信や公開 API でよく使われる。

---

## キャッシュ・条件付きリクエスト

キャッシュ可能なレスポンスには `Cache-Control` を付ける。更新頻度が低いリソースでは `ETag` や `Last-Modified` を使うと、再取得時の通信量を減らせる。

```http
Cache-Control: public, max-age=300
ETag: "user-u_123-v3"
```

クライアントは `If-None-Match` を送信し、変更がなければサーバーは `304 Not Modified` を返す。

```http
If-None-Match: "user-u_123-v3"
```

更新時に `If-Match` を使うと、古いデータでの上書きを防げる。

```http
If-Match: "user-u_123-v3"
```

---

## レート制限

リクエスト数を制限する場合は `429 Too Many Requests` を返す。再試行可能なタイミングを伝えるために `Retry-After` や `RateLimit-*` ヘッダーを付ける。

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60
RateLimit-Limit: 100
RateLimit-Remaining: 0
RateLimit-Reset: 60
```

---

## 非同期処理

時間がかかる処理は同期レスポンスで完了を待たず、`202 Accepted` とジョブリソースを返す設計にする。

```http
POST /exports
```

```http
HTTP/1.1 202 Accepted
Location: /jobs/job_123
Retry-After: 10
```

クライアントはジョブの状態をポーリングする。

```http
GET /jobs/job_123
```

```json
{
  "id": "job_123",
  "status": "running",
  "progress": 40
}
```

---

## OpenAPI（Swagger）

OpenAPI 3.1.x の YAML 形式。

### 基本構造

```yaml
# OpenAPI のバージョン
openapi: 3.1.0

# API 全体のメタ情報
info:
  # API 名
  title: My API
  # API 仕様書のバージョン。URL の /v1 とは別物
  version: 1.0.0
  # API の概要説明
  description: API の説明

# API のベース URL
servers:
  - url: https://api.example.com/v1
    description: Production
  - url: http://localhost:8080/v1
    description: Local

# エンドポイント一覧
paths:
  # /users に対する操作
  /users:
    # GET /users
    get:
      # API ドキュメントに表示される短い説明
      summary: ユーザー一覧取得
      ...
    # POST /users
    post:
      summary: ユーザー作成
      ...
```

### パス定義

```yaml
paths:
  # {id} はパスパラメータ。parameters で定義する
  /users/{id}:
    get:
      # API ドキュメントに表示される短い説明
      summary: ユーザー取得
      # クライアント生成などで使われる一意な操作 ID
      operationId: getUser
      # ドキュメント上の分類
      tags:
        - Users
      # パス、クエリ、ヘッダーなどのパラメータ
      parameters:
        - name: id
          # path は URL パス内のパラメータ
          in: path
          # path パラメータは必須
          required: true
          # パラメータの型
          schema:
            type: string
        - name: includePosts
          # query は ?includePosts=true のようなクエリパラメータ
          in: query
          # query パラメータは任意にできる
          required: false
          schema:
            type: boolean
            default: false
        - name: X-Request-Id
          # header は HTTP ヘッダー
          in: header
          required: false
          schema:
            type: string
          description: リクエスト追跡用 ID
      # ステータスコードごとのレスポンス定義
      responses:
        "200":
          description: 成功
          # レスポンスボディのメディアタイプ
          content:
            application/json:
              schema:
                # components/schemas/User を参照
                $ref: "#/components/schemas/User"
        "404":
          description: 存在しない
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"

    patch:
      summary: ユーザー更新
      operationId: updateUser
      tags:
        - Users
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      # リクエストボディ
      requestBody:
        required: true
        content:
          application/json:
            schema:
              # 更新用リクエストスキーマを参照
              $ref: "#/components/schemas/UpdateUserRequest"
      responses:
        "200":
          description: 更新成功
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/User"
```

### スキーマ定義

```yaml
components:
  # 再利用可能なデータ構造
  schemas:
    User:
      # JSON オブジェクト
      type: object
      # 必須フィールド
      required:
        - id
        - name
        - email
      # フィールド定義
      properties:
        id:
          type: string
          # サンプル値
          example: "u_123"
        name:
          type: string
          example: "Ken Ishikawa"
        email:
          type: string
          # email 形式としてバリデーション・ドキュメント化される
          format: email
          example: "ken@example.com"
        createdAt:
          type: string
          # ISO 8601 の日時形式
          format: date-time
          # レスポンス専用フィールド
          readOnly: true

    UpdateUserRequest:
      type: object
      properties:
        name:
          type: string
        email:
          type: string
          format: email

    Error:
      type: object
      required:
        - title
        - status
      properties:
        title:
          type: string
        status:
          type: integer
        detail:
          type: string

    # 配列レスポンス例
    UserList:
      type: object
      properties:
        data:
          type: array
          # 配列要素の型
          items:
            $ref: "#/components/schemas/User"
        pagination:
          $ref: "#/components/schemas/Pagination"

    Pagination:
      type: object
      properties:
        total:
          type: integer
        page:
          type: integer
        perPage:
          type: integer
        nextCursor:
          type: string
          # null を許可。次ページがない場合などに使う
          nullable: true
          description: 次ページ取得用の opaque cursor
          example: "eyJjcmVhdGVkQXQiOiIyMDI2LTA1LTI1VDEwOjAwOjAwWiIsImlkIjoidV8xMjQifQ"
```

### 認証の定義

```yaml
components:
  # 認証方式の定義
  securitySchemes:
    BearerAuth:
      # HTTP Authorization ヘッダーを使う認証
      type: http
      # Authorization: Bearer <token>
      scheme: bearer
      # ドキュメント上の補足。JWT に限定するものではない
      bearerFormat: JWT
    ApiKey:
      # API キー認証
      type: apiKey
      # ヘッダーで API キーを送る
      in: header
      # ヘッダー名
      name: X-API-Key

# グローバルに適用
security:
  # すべてのエンドポイントに BearerAuth を要求
  - BearerAuth: []

# エンドポイント個別に上書き可
paths:
  /public/health:
    get:
      security: []   # 認証不要
```

### 運用

OpenAPI はドキュメントだけでなく、実装・テスト・クライアント生成の基準として扱う。

- lint で命名規則、必須レスポンス、認証定義をチェックする
- API 変更時に OpenAPI の差分をレビューする
- 破壊的変更がないか後方互換性チェックを行う
- SDK や型定義を生成する場合は、operationId を安定させる
- サンプルレスポンスと実レスポンスがずれないようにテストする

---

## 設計のベストプラクティス

### 命名規則

| 対象           | 推奨            | 例                    |
|----------------|-----------------|-----------------------|
| URL パス       | `kebab-case`    | `/user-profiles`      |
| クエリパラメータ| `camelCase`     | `?sortBy=createdAt`   |
| JSON フィールド | `camelCase`     | `"createdAt"`         |
| OpenAPI operationId | `camelCase` | `getUserById`        |

### 互換性

既存クライアントが壊れる変更は破壊的変更として扱う。

**比較的安全な変更**

- 任意フィールドを追加する
- 新しいエンドポイントを追加する
- enum に新しい値を追加する（クライアントが未知の値を許容する場合）
- レスポンスに省略可能なメタデータを追加する

**破壊的変更**

- URL、HTTP メソッド、認証方式を変更する
- 必須リクエストフィールドを追加する
- レスポンスフィールドを削除・リネームする
- フィールドの型や意味を変える
- ステータスコードやエラー形式を互換性なく変える

破壊的変更が必要な場合は、新しいバージョンを用意し、旧バージョンの廃止予定日を明示する。

### 安全性と冪等性

| メソッド | 安全 | 冪等 | 理由                                       |
|----------|------|------|--------------------------------------------|
| GET      | ○    | ○    | 取得のみ。サーバー状態を変更しない         |
| HEAD     | ○    | ○    | ヘッダーのみ取得する                       |
| OPTIONS  | ○    | ○    | 利用可能な通信オプションを取得する         |
| PUT      | ✗    | ○    | 同じリクエストなら最終状態が同じ           |
| DELETE   | ✗    | ○    | 再実行してもリソースが削除済みという最終状態は同じ |
| POST     | ✗    | ✗    | 毎回新規リソースを作成する可能性           |
| PATCH    | ✗    | △    | 内容によって冪等にも非冪等にもなる         |

安全なメソッドはサーバー状態を変更しない。冪等なメソッドは同じ操作を複数回行っても最終状態が変わらない。

冪等な操作はネットワークエラー時にリトライしやすい。

### リトライと Idempotency-Key

`POST` は通常は非冪等だが、クライアントが同じリクエストで同じ `Idempotency-Key` を送ることで、サーバー側で重複処理を防げる。

```http
POST /payments
Idempotency-Key: 7b8f2b6d-8c22-4f1b-8d27-8d8f4b35a111
```

決済、注文作成、メール送信など、二重実行が問題になる処理で有効。

### PATCH の形式

`PATCH` のリクエスト形式は API ごとに明確に決める。

**JSON Merge Patch** はオブジェクトの差分を送る。`null` はフィールド削除を意味することがある。

```http
Content-Type: application/merge-patch+json
```

```json
{
  "name": "Ken Ishikawa"
}
```

**JSON Patch** は操作の配列を送る。

```http
Content-Type: application/json-patch+json
```

```json
[
  { "op": "replace", "path": "/name", "value": "Ken Ishikawa" }
]
```

### HATEOAS

HATEOAS（Hypermedia as the Engine of Application State）は、レスポンスにリンクを含めることで、クライアントが次のアクションを発見できるようにする考え方。

Richardson Maturity Model は REST API の設計成熟度を段階的に説明するモデル。

| レベル | 内容                                      | 例                                      |
|--------|-------------------------------------------|-----------------------------------------|
| Level 0 | HTTP を単なる通信路として使う             | `POST /api` にすべての操作を送る        |
| Level 1 | リソースごとに URL を分ける               | `/users`, `/orders/{id}`                |
| Level 2 | HTTP メソッドとステータスコードを使う     | `GET /users`, `POST /orders`, `404`     |
| Level 3 | レスポンスに次の操作リンクを含める        | `_links.cancel`, `_links.payment`       |

HATEOAS は Level 3 にあたる。

```json
{
  "id": "o_456",
  "status": "pending",
  "_links": {
    "self":    { "href": "/orders/o_456" },
    "cancel":  { "href": "/orders/o_456/cancel", "method": "POST" },
    "payment": { "href": "/orders/o_456/payment", "method": "POST" }
  }
}
```

実務では Level 2（メソッド + ステータスコード）までの実装が多い。

---

## 用語集

| 用語           | 説明                                                    |
|----------------|---------------------------------------------------------|
| REST           | Representational State Transfer。HTTP を利用した設計スタイル |
| OpenAPI        | REST API の仕様を記述する標準フォーマット（旧 Swagger）  |
| CRUD           | Create / Read / Update / Delete の頭字語               |
| 安全性         | HTTP メソッドがサーバー状態を変更しない性質             |
| 冪等性         | 同じ操作を複数回行っても最終状態が変わらない性質        |
| JWT            | JSON Web Token。JSON 形式で表現されたトークン形式       |
| opaque token   | クライアントから中身を解釈できないトークン              |
| ETag           | リソースのバージョンや内容を表す識別子                  |
| Idempotency-Key| 重複リクエストを同一操作として扱うためのキー            |
| ページネーション| 大量データを分割して返す仕組み                          |
| HATEOAS        | レスポンスにリンクを含めて次のアクションを示す設計原則  |
| リソース       | API が操作する対象（ユーザー、記事など）                |
| エンドポイント | API の特定 URL（`/users/{id}` など）                    |

---

## リンク集

- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Swagger Editor](https://editor.swagger.io/) — OpenAPI を書いて即プレビュー
- [RFC 9457 — Problem Details](https://www.rfc-editor.org/rfc/rfc9457)
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111)
- [RFC 9457 Info](https://www.rfc-editor.org/info/rfc9457)
- [REST API Tutorial](https://restfulapi.net/)
- [HTTP Status Codes](https://developer.mozilla.org/ja/docs/Web/HTTP/Status)

# MFクラウド請求書APIドキュメント
## versions

|version|link|
|:--:|:--:|
|v1|https://github.com/nishisuke/invoice-api-doc/tree/v1|

## Resources
- [見積書](/quotes)
- [事業所](/office)
- [取引先](/partners)
- [品目](/items)

## プランごとの利用制限について
どのプランでも下記制限の中でAPIを利用することができます。

| プラン     | 請求書作成リクエスト上限 |
| :--        | --:                      |
| Free       | 100                      |
| Basic      | 100                      |
| Pro        | なし                     |
| Enterprise | なし                     |

## 認証について
アプリケーションの認証はOAuth2.0のAuthorization Code Grantにもとづいて行います。

### アプリケーションの登録
* 画面右上の歯車のアイコンをクリックし、そこからAPI連携を選びます
* 新規作成ボタンをクリックし、フォームに必要な情報を入力し、利用規約に同意する、にチェックを入れて作成ボタンをクリックします
* Client IDとClient Secretが発行されます
 redirect_uri は https のみ許可しています。

### アクセストークンの発行
* 前項で発行した Client IDとアプリケーション作成時に入力した値を使い、下記のようなURLにアクセスします。
```
https://invoice.moneyforward.com/oauth/authorize?client_id=[CLIENT_ID]&redirect_uri=[REDIRECT_URL]&response_type=code&scope=[SCOPE]
```
* アプリケーションを承認すると認証コードつきURLが発行されるので、そのコードを使って以下のようなリクエストをサーバーに発行し、アクセストークンを取得します。入力に使う[REDIRECT_URL]はアプリケーションの作成時に入力した値を入れてください

```
curl -d client_id=[CLIENT_ID] -d client_secret=[CLIENT_SECRET] -d redirect_uri=[REDIRECT_URL] -d grant_type=authorization_code -d code=[認証コード] -X POST https://invoice.moneyforward.com/oauth/token
```

※リクエストを送る際、下記2点について注意して下さい。

* パラメータは `Content-Type: application/x-www-form-urlencoded` にする
* クライアントによっては `Accept: application/json` ヘッダーの明示が必要

### アクセストークンの再発行

発行されたアクセストークンの有効期限は、30日間です。
期限切れアクセストークンの再発行については[MFクラウド請求書API スタートアップガイド](https://support.biz.moneyforward.com/invoice/guide/api-guide/a01.html)に記載していますのでそちらをご覧ください。

## アクセス数の制限について
* Basicプラン以下のプランをご利用の場合、月のAPI経由での請求書作成数を100件までとさせていただきます。
* その他、取引先登録数や郵送の可否等は通常のプランごとの制限と同様* となります。

## クライアントライブラリ

* [Ruby](https://github.com/moneyforward/mf_cloud-invoice-ruby)


## 請求書API
### エンドポイント
```
  /api/v1/billings
```

### 請求書一覧の取得
```
  GET /api/v1/billings.json
```

#### パラメーター
| 名称                  | field    | 備考 |
| :--                   | :--      | :--|
| ページ番号            | page     | |
| 1ページあたりの表示数 | per_page | 100件まで |

#### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" "https://invoice.moneyforward.com/api/v1/billings.json?page=1&per_page=10"
```

#### レスポンス
```
HTTP/1.1 200 OK

{
  "meta" : {
    "total_count" : 1,
    "total_pages" : 1,
    "current_page" : 1,
    "per_page" : 10
  },
  "billings" : [
    {
      "id" : "ABCDEFGHIJKLMNOPQRST123",
      "user_id" : "ABCDEFGHIJKLMNOPQRST456",
      "partner_id" : "ABCDEFGHIJKLMNOPQRST789",
      "department_id" : "ABCDEFGHIJKLMNOPQRST012",
      "partner_name" : "サンプル取引先",
      "partner_name_suffix" : "様",
      "partner_detail" : "hogehoge",
      "member_id" : "ABCDEFGHIJKLMNOPQRST345",
      "member_name" : "担当者A",
      "office_name" : "サンプル事業所",
      "office_detail" : "",
      "title" : "件名サンプル",
      "excise_price" : 80,
      "deduct_price" : 0,
      "subtotal" : 1000,
      "memo" : "",
      "payment_condition" : "",
      "total_price" : 1080,
      "billing_date" : "2015/10/31",
      "due_date" : "2015/11/30",
      "sales_date" : "2015/10/31",
      "created_at" : "2015/10/31T00:00:00.000+09:00",
      "updated_at" : "2015/10/31T00:00:00.000+09:00",
      "billing_number": "1",
      "note": "",
      "document_name": "",
      "tags": [
        "tag1",
        "tag2"
      ],
      "status" : {
        "posting" : "未郵送",
        "email" : "未送信",
        "download" : "",
        "payment" : "未設定"
      },
      "items" : [
        {
          "id" : "ABCDEFGHIJKLMNOPQRST012",
          "code" : "ITEM-001",
          "name" : "商品A",
          "detail" : "",
          "quantity" : 1,
          "unit_price" : 1000,
          "unit" : "個",
          "price" : 1000,
          "display_order" : 0,
          "excise": true,
          "created_at" : "2015/10/31T00:00:00.000+09:00",
          "updated_at" : "2015/10/31T00:00:00.000+09:00"
        }
      ]
    }
  ]
}
```

### 請求書一覧の検索
```
  GET /api/v1/billings/search.json
```

#### パラメーター
| 名称                  | field     | 備考 |
| :--                   | :--       | :-- |
| ページ番号            | page      | |
| 1ページあたりの表示数 | per_page  | 100件まで |
| 検索文字列            | q         |  取引先(完全一致), ステータス、件名etc |
| 期間絞込対象          | range_key |  created_at(デフォルト), sales_date, billing_dateのみ許可 |
| 期間開始日            | from      |  デフォルト: 月初 |
| 期間終了日            | to        |  デフォルト: 月末 |

#### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" "https://invoice.moneyforward.com/api/v1/billings/search.json?q=hoge&range_key=created_at&from=2015-10-01&to=2015-10-31"
```

#### レスポンス
```
HTTP/1.1 200 OK

{
  "meta" : {
    "total_count" : 1,
    "total_pages" : 1,
    "current_page" : 1,
    "per_page" : 100,
    "condition" : {
      "query" : "hoge",
      "range_key" : "created_at",
      "from" : "2015-10-01",
      "to" : "2015-10-31"
    }
  },
  "billings" : [
    {
      "id" : "ABCDEFGHIJKLMNOPQRST123",
      "user_id" : "ABCDEFGHIJKLMNOPQRST456",
      "partner_id" : "ABCDEFGHIJKLMNOPQRST789",
      "department_id" : "ABCDEFGHIJKLMNOPQRST012",
      "partner_name" : "サンプル取引先",
      "partner_name_suffix" : "様",
      "partner_detail" : "hogehoge",
      "member_id" : "ABCDEFGHIJKLMNOPQRST345",
      "member_name" : "担当者A",
      "office_name" : "サンプル事業所",
      "office_detail" : "",
      "title" : "件名サンプル",
      "excise_price" : 80,
      "deduct_price" : 0,
      "subtotal" : 1000,
      "memo" : "",
      "payment_condition" : "",
      "total_price" : 1080,
      "payment_condition" : "",
      "billing_date" : "2015/10/31",
      "due_date" : "2015/11/30",
      "sales_date" : "2015/10/31",
      "created_at" : "2015/10/31T00:00:00.000+09:00",
      "updated_at" : "2015/10/31T00:00:00.000+09:00",
      "billing_number" : "1",
      "note" : "",
      "document_name" : "",
      "tags": [
        "tag1",
        "tag2"
      ],
      "status" : {
        "posting" : "未郵送",
        "email" : "未送信",
        "download" : "",
        "payment" : "未設定"
      },
      "items" : [
        {
          "id" : "ABCDEFGHIJKLMNOPQRST012",
          "code" : "ITEM-001",
          "name" : "商品A",
          "detail" : "",
          "quantity" : 1,
          "unit_price" : 1000,
          "unit" : "個",
          "price" : 1000,
          "excise": true,
          "display_order" : 0,
          "created_at" : "2015/10/31T00:00:00.000+09:00",
          "updated_at" : "2015/10/31T00:00:00.000+09:00"
        }
      ]
    }
  ]
}
```

### 請求書の取得
```
  GET /api/v1/billings/:id.json
```

#### パラメーター
なし

#### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" https://invoice.moneyforward.com/api/v1/billings/ABCDEFGHIJKLMNOPQRST123.json
```

#### レスポンス
```
HTTP/1.1 200 OK

{
  "id" : "ABCDEFGHIJKLMNOPQRST123",
  "pdf_url" : "https://invoice.moneyforward.co.jp/api/v1/billings/:id.pdf",
  "user_id" : "ABCDEFGHIJKLMNOPQRST456",
  "partner_id" : "ABCDEFGHIJKLMNOPQRST789",
  "department_id" : "ABCDEFGHIJKLMNOPQRST012",
  "partner_name" : "サンプル取引先",
  "partner_name_suffix" : "様",
  "partner_detail" : "hogehoge",
  "member_id" : "ABCDEFGHIJKLMNOPQRST345",
  "member_name" : "担当者A",
  "office_name" : "サンプル事業所",
  "office_detail" : "",
  "title" : "件名サンプル",
  "excise_price" : 80,
  "deduct_price" : 0,
  "subtotal" : 1000,
  "memo" : "",
  "payment_condition" : "",
  "total_price" : 1080,
  "payment_condition" : "",
  "billing_date" : "2015/10/31",
  "due_date" : "2015/11/30",
  "sales_date" : "2015/10/31",
  "created_at" : "2015/10/31T00:00:00.000+09:00",
  "updated_at" : "2015/10/31T00:00:00.000+09:00",
  "billing_number" : "1",
  "note" : "",
  "document_name" : "",
  "tags": [
    "tag1",
    "tag2"
  ],
  "status" : {
    "posting" : "未郵送",
    "email" : "未送信",
    "download" : "",
    "payment" : "未設定"
  },
  "items" : [
    {
      "id" : "ABCDEFGHIJKLMNOPQRST012",
      "name" : "商品A",
      "code" : "ITEM-001",
      "detail" : "",
      "quantity" : 1,
      "unit_price" : 1000,
      "unit" : "個",
      "price" : 1000,
      "excise": true,
      "display_order" : 0,
      "created_at" : "2015/10/31T00:00:00.000+09:00",
      "updated_at" : "2015/10/31T00:00:00.000+09:00"
    }
  ]
}
```

#### エラーレスポンス
```
{
  "code" : "404",
  "errors" : [
    {
      "message" : "存在しないIDが渡されました。"
    }
  ]
}
```

### 請求書pdfの取得
```
  GET /api/v1/billings/:id.pdf
```

#### パラメーター
なし

#### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" https://invoice.moneyforward.com/api/v1/billings/ABCDEFGHIJKLMNOPQRST123.pdf -O
```

#### レスポンス
pdfファイルがバイナリで送信されます。
```
HTTP/1.1 200 OK
Content-Type: application/pdf
```

#### エラーレスポンス
```
HTTP/1.1 404 Not Found

{
  "code" : "404",
  "errors" : [
    {
      "message" : "存在しないIDが渡されました。"
    }
  ]
}
```

### 請求書の作成
```
  POST /api/v1/billings
```

#### パラメーター
| 名称         | field                      | 備考 |
| :--          | :--                        | :-- |
| 部門ID       | billing[department_id]     | 必須 |
| 件名         | billing[title]             | |
| 請求書番号   | billing[billing_number]    | |
| 振込先       | billing[payment_condition] | |
| 備考         | billing[note]              | |
| 請求日       | billing[billing_date]      | |
| お支払期限   | billing[due_date]          | |
| 売上計上日   | billing[sales_date]        | |
| メモ         | billing[memo]              | |
| 帳票名       | billing[document_name]     | |
| タグ         | billing[tags]              | カンマ区切り文字列で記載 |
| 品目1 名前   | items[0][name]             | |
| 品目1 コード | items[0][code]             | |
| 品目1 詳細   | items[0][detail]           | |
| 品目1 数量   | items[0][quantity]         | |
| 品目1 単価   | items[0][unit_price]       | |
| 品目1 単位   | items[0][unit]             | |
| 品目1 税対象 | items[0][excise]           | 0: false  1: true |
| 品目2 名前   | items[1][name]             | |
| 品目2 コード | items[1][code]             | |
| 品目2 詳細   | items[1][detail]           | |
| 品目2 数量   | items[1][quantity]         | |
| 品目2 単価   | items[1][unit_price]       | |
| 品目2 単位   | items[1][unit]             | |
| 品目2 税対象 | items[1][excise]           | 0: false  1: true |

#### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" -H "Content-Type: application/json" -d '{ "billing" : { "department_id" : "DEPARTMENT_ID" }}' -X POST https://invoice.moneyforward.com/api/v1/billings

# 品目を含むリクエスト
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" -H "Content-Type: application/json" \
-d '
{
  "billing" : {
    "department_id" : "DEPARTMENT_ID",
      "items" : [
        {
          "name" : "商品A",
          "quantity" : "1",
          "unit_price" : "100"
        },
        # 登録した品目を利用する場合
        {
          "code" : "登録済み品目コード"
        }
      ]
  }
} '
-X POST https://invoice.moneyforward.com/api/v1/billings

# タグを含むリクエスト
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" -H "Content-Type: application/json" \
-d '
{
  "billing" : {
    "department_id" : "DEPARTMENT_ID",
      "items" : [
        {
          "name" : "商品A",
          "quantity" : "1",
          "unit_price" : "100"
        }
      ],
    "tags" : "tag1,tag2" # カンマ区切り文字列で入力
  }
} '
-X POST https://invoice.moneyforward.com/api/v1/billings
```
#### レスポンス
```
HTTP/1.1 201 Created

{
  "id" : "ABCDEFGHIJKLMNOPQRST123",
  "pdf_url" : "https://invoice.moneyforward.co.jp/api/v1/billings/:id.pdf",
  "user_id" : "ABCDEFGHIJKLMNOPQRST456",
  "partner_id" : "ABCDEFGHIJKLMNOPQRST789",
  "department_id" : "ABCDEFGHIJKLMNOPQRST012",
  "partner_name" : "サンプル取引先",
  "partner_name_suffix" : "様",
  "partner_detail" : "hogehoge",
  "member_id" : "ABCDEFGHIJKLMNOPQRST345",
  "member_name" : "担当者A",
  "office_name" : "サンプル事業所",
  "office_detail" : "",
  "title" : "件名サンプル",
  "excise_price" : 80,
  "deduct_price" : 0,
  "subtotal" : 1000,
  "memo" : "",
  "payment_condition" : "",
  "total_price" : 1080,
  "billing_date" : "2015/10/31",
  "due_date" : "2015/11/30",
  "sales_date" : "2015/10/31",
  "created_at" : "2015/10/31T00:00:00.000+09:00",
  "updated_at" : "2015/10/31T00:00:00.000+09:00",
  "billing_number" : "1",
  "note" : "",
  "document_name" : "",
  "tags": [
    "tag1",
    "tag2"
  ],
  "status" : {
    "posting" : "未郵送",
    "email" : "未送信",
    "download" : "",
    "payment" : "未設定"
  },
  "items" : [
    {
      "id" : "ABCDEFGHIJKLMNOPQRST012",
      "name" : "商品A",
      "detail" : "",
      "quantity" : 1,
      "unit_price" : 1000,
      "unit" : "個",
      "price" : 1000
      "excise": true,
      "display_order" : 0,
      "created_at" : "2015/10/31T00:00:00.000+09:00",
      "updated_at" : "2015/10/31T00:00:00.000+09:00"
    }
  ]
}
```

#### エラーレスポンス
department_idが不正であった場合
```
{
  "code" : "404",
  "errors" : [
    {
      "message" : "存在しないIDが渡されました。"
    }
  ]
}
```

### 請求書の更新
```
  PATCH /api/v1/billings/:id
```
#### パラメーター
| 名称         | field                      | 備考 |
| :--          | :--                        | :-- |
| 部門ID       | billing[department_id]     | |
| 件名         | billing[title]             | |
| 請求書番号   | billing[billing_number]    | |
| 振込先       | billing[payment_condition] | |
| 備考         | billing[note]              | |
| 請求日       | billing[billing_date]      | |
| お支払期限   | billing[due_date]          | |
| 売上計上日   | billing[sales_date]        | |
| メモ         | billing[memo]              | |
| 帳票名       | billing[document_name]     | |
| タグ         | billing[tags]              | カンマ区切り文字列で記載 |
| 品目1 名前   | items[0][name]             | |
| 品目1 コード | items[0][code]             | |
| 品目1 詳細   | items[0][detail]           | |
| 品目1 数量   | items[0][quantity]         | |
| 品目1 単価   | items[0][unit_price]       | |
| 品目1 単位   | items[0][unit]             | |
| 品目1 税対象 | items[0][excise]           | 0: false  1: true |
| 品目2 名前   | items[1][name]             | |
| 品目2 コード | items[1][code]             | |
| 品目2 詳細   | items[1][detail]           | |
| 品目2 数量   | items[1][quantity]         | |
| 品目2 単価   | items[1][unit_price]       | |
| 品目2 単位   | items[1][unit]             | |
| 品目2 税対象 | items[1][excise]           | 0: false  1: true |

#### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" -H "Content-Type: application/json" -d '{ "billing" : { "title" : "件名サンプル" }}' -X PATCH https://invoice.moneyforward.com/api/v1/billings/ABCDEFGHIJKLMNOPQRST123

# 品目を含むリクエスト
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" -H "Content-Type: application/json" \
-d '
{
  "billing" : {
    "department_id" : "DEPARTMENT_ID",
      "items" : [
        # itemの追加(idなし)
        {
          "name" : "商品C",
          "quantity" : "1",
          "unit_price" : "100"
        },
        # itemの更新(idあり)
        {
          "id" : "ABCDEFGHIJKLMNOPQRST012"
          "name" : "商品B",
          "quantity" : "1",
          "unit_price" : "100"
        },
        # itemの更新(削除)
        {
          "id" : "ABCDEFGHIJKLMNOPQRST012"
          "_destroy" : true
        },
      ]
  }
} '
-X PATCH https://invoice.moneyforward.com/api/v1/billings/"更新する請求書ID"
```

#### レスポンス
```
HTTP/1.1 200 OK

{
  "id" : "ABCDEFGHIJKLMNOPQRST123",
  "pdf_url" : "https://invoice.moneyforward.co.jp/api/v1/billings/:id.pdf",
  "user_id" : "ABCDEFGHIJKLMNOPQRST456",
  "partner_id" : "ABCDEFGHIJKLMNOPQRST789",
  "department_id" : "ABCDEFGHIJKLMNOPQRST012",
  "partner_name" : "サンプル取引先",
  "partner_name_suffix" : "様",
  "partner_detail" : "hogehoge",
  "member_id" : "ABCDEFGHIJKLMNOPQRST345",
  "member_name" : "担当者A",
  "office_name" : "サンプル事業所",
  "office_detail" : "",
  "title" : "件名サンプル",
  "excise_price" : 80,
  "deduct_price" : 0,
  "subtotal" : 1000,
  "memo" : "",
  "payment_condition" : "",
  "total_price" : 1080,
  "billing_date" : "2015/10/31",
  "due_date" : "2015/11/30",
  "sales_date" : "2015/10/31",
  "created_at" : "2015/10/31T00:00:00.000+09:00",
  "updated_at" : "2015/10/31T00:00:00.000+09:00",
  "billing_number" : "1",
  "note" : "",
  "document_name" : "",
  "tags": [
    "tag1",
    "tag2"
  ],
  "status" : {
    "posting" : "未郵送",
    "email" : "未送信",
    "download" : "",
    "payment" : "未設定"
  },
  "items" : [
    {
      "id" : "ABCDEFGHIJKLMNOPQRST012",
      "code" : "ITEM-001",
      "name" : "商品A",
      "detail" : "",
      "quantity" : 1,
      "unit_price" : 1000,
      "unit" : "個",
      "price" : 1000,
      "excise": true,
      "display_order" : 0,
      "created_at" : "2015/10/31T00:00:00.000+09:00",
      "updated_at" : "2015/10/31T00:00:00.000+09:00"
    }
  ]
}
```

#### エラーレスポンス
IDが不正である場合
```
HTTP/1.1 404 Not Found

{
  "code" : "404",
  "errors" : [
    {
      "message" : "存在しないIDが渡されました。"
    }
  ]
}
```

### 請求書の入金ステータスの更新
```
  PATCH /api/v1/billings/:id/billing_status/payment
```
#### パラメーター
| 名称           | field                   | 必須 or 任意 | 備考                                        |
| :--            | :--                     | :--          | :--                                         |
| 入金ステータス | billing_status[payment] | 必須         | '0': 未設定, '1': '未入金', '2': '入金済み' |

#### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" -H "Content-Type: application/json" -d '{ "billing_status" : { "payment" : "0"} }' -X PATCH https://invoice.moneyforward.com/api/v1/billings/ABCDEFGHIJKLMNOPQRST123/billing_status/payment
```

#### レスポンス
```
HTTP/1.1 200 OK

{
  "id" : "ABCDEFGHIJKLMNOPQRST123",
  "pdf_url" : "https://invoice.moneyforward.co.jp/api/v1/billings/:id.pdf",
  "user_id" : "ABCDEFGHIJKLMNOPQRST456",
  "partner_id" : "ABCDEFGHIJKLMNOPQRST789",
  "department_id" : "ABCDEFGHIJKLMNOPQRST012",
  "partner_name" : "サンプル取引先",
  "partner_name_suffix" : "様",
  "partner_detail" : "hogehoge",
  "member_id" : "ABCDEFGHIJKLMNOPQRST345",
  "member_name" : "担当者A",
  "office_name" : "サンプル事業所",
  "office_detail" : "",
  "title" : "件名サンプル",
  "excise_price" : 80,
  "deduct_price" : 0,
  "subtotal" : 1000,
  "memo" : "",
  "payment_condition" : "",
  "total_price" : 1080,
  "billing_date" : "2015/10/31",
  "due_date" : "2015/11/30",
  "sales_date" : "2015/10/31",
  "created_at" : "2015/10/31T00:00:00.000+09:00",
  "updated_at" : "2015/10/31T00:00:00.000+09:00",
  "billing_number" : "1",
  "note" : "",
  "document_name" : "",
  "tags": [
    "tag1",
    "tag2"
  ],
  "status" : {
    "posting" : "未郵送",
    "email" : "未送信",
    "download" : "",
    "payment" : "未設定"
  },
  "items" : [
    {
      "id" : "ABCDEFGHIJKLMNOPQRST012",
      "code" : "ITEM-001",
      "name" : "商品A",
      "detail" : "",
      "quantity" : 1,
      "unit_price" : 1000,
      "unit" : "個",
      "price" : 1000,
      "excise": true,
      "display_order" : 0,
      "created_at" : "2015/10/31T00:00:00.000+09:00",
      "updated_at" : "2015/10/31T00:00:00.000+09:00"
    }
  ]
}
```

#### エラーレスポンス
IDが不正である場合
```
HTTP/1.1 404 Not Found

{
  "code" : "404",
  "errors" : [
    {
      "message" : "存在しないIDが渡されました。"
    }
  ]
}
```

不正な帳票を指定した場合
```
HTTP/1.1 400 Bad Request

{
  "code" : "400",
  "errors" : [
    {
      "message" : "受け取った帳票のステータスは変更できません。"
    }
  ]
}
```

不正なパラメータを指定した場合
```
HTTP/1.1 400 Bad Request

{
  "code" : "400",
  "errors" : [
    {
      "message" : "ステータスの値が不正です。"
    }
  ]
}
```

### 請求書の郵送
```
  POST /api/v1/billings/:id/posting
```

#### パラメーター
なし

#### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" -H "Content-Type: application/json" -d '' -X POST https://invoice.moneyforward.com/api/v1/billings/ABCDEFGHIJKLMNOPQRST123/posting
```

#### レスポンス
```
HTTP/1.1 200 OK
```

#### エラーレスポンス
既に郵送済みである場合
```
HTTP/1.1 400 Bad Request

{
  "code" : "400",
  "errors" : [
    {
      "message" : "既に郵送済みです"
    }
  ]
}
```

### 請求書の郵送キャンセル
```
  POST /api/v1/billings/:id/cancel_posting
```

#### パラメーター
なし

#### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" -H "Content-Type: application/json" -d '' -X POST https://invoice.moneyforward.com/api/v1/billings/ABCDEFGHIJKLMNOPQRST123/cancel_posting
```

#### レスポンス
```
HTTP/1.1 200 OK
```

### 請求書の削除
```
  DELETE /api/v1/billings/:id
```

#### パラメーター
なし

#### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" -X DELETE https://invoice.moneyforward.com/api/v1/billings/ABCDEFGHIJKLMNOPQRST123
```

#### レスポンス
```
HTTP/1.1 204 No Content
```

#### エラーレスポンス
IDが不正である場合
```
HTTP/1.1 404 Not Found

{
  "code" : "404",
  "errors" : [
    {
      "message" : "存在しないIDが渡されました。"
    }
  ]
}
```


## 送付履歴API
### エンドポイント
```
  /api/v1/sent_history
```

### 送付履歴一覧の取得
```
  GET /api/v1/sent_history.json
```

#### パラメーター
| 名称                  | field    | 備考 |
| :--                   | :--      | :--|
| ページ数              | page     | |
| 1ページあたりの表示数 | per_page | 100件まで |

#### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" "https://invoice.moneyforward.com/api/v1/sent_history.json?page=1&per_page=10"
```

#### レスポンス
HTTP/1.1 200 OK

```
{
  "meta": {
    "total_count" : 1,
    "total_pages" : 1,
    "current_page" : 1,
    "per_page" : 10
  },
  "sent_history_list": [
    {
      "operator_id":  "ABCDEFGHIJKLMNOP",
      "type": "メール",
      "document_type": "請求書",
      "document_id": "ABCDEFGHIJKLMNOP",
      "from": "",
      "to": "sample@moneyforward.co.jp",
      "cc": "",
      "sent_at": "2015-05-15T11:40:44.000+09:00"
    }
  ]
}
```

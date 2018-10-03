### 取引先の作成
```
  POST /api/v1/partners
```

#### パラメーター
| 名称             | field                    | 備考 |
| :--              | :--                      | :--|
| 顧客コード       | partner[code]            | |
| 名前             | partner[name]            | 必須 |
| 名前（カナ）     | partner[name_kana]       | |
| 敬称             | partner[name_suffix]     | |
| メモ             | partner[memo]            | | |
| 郵便番号         | partner[zip]             | |
| 電話番号         | partner[tel]             | |
| 都道府県         | partner[prefecture]      | 入力なしの場合、東京都 |
| 住所1            | partner[address1]        | |
| 住所2            | partner[address2]        | |
| 担当者氏名       | partner[person_name]     | |
| 担当者役職       | partner[person_title]    | |
| 部門名           | partner[department_name] | |
| メールアドレス   | partner[email]           | |
| ccメールアドレス | partner[cc_emails]       | カンマ区切りで複数入力可 |

#### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" -H "Content-Type: application/json" -d '{ "partner" : { "name" : "サンプル取引先", "code" : "ABCD-00001" }}' -X POST https://invoice.moneyforward.com/api/v1/partners
```

#### レスポンス
※取引先を作成すると、自動で部門(department)が作成されます。請求書作成時などではdepartment_のidを使用します。
```
HTTP/1.1 201 Created

{
  "id" : "ABCDEFGHIJKLMNOPQRST123",
  "code" : "ABCD-00001",
  "name" : "サンプル取引先",
  "name_kana" : "サンプルトリヒキサキ",
  "name_suffix" : "様",
  "memo" : "",
  "created_at" : "2015/10/31T00:00:00.000+09:00",
  "updated_at" : "2015/10/31T00:00:00.000+09:00",
  "departments" : [
    {
      "id" : "ABCDEFGHIJKLMNOPQRST012",
      "name" : "開発部",
      "zip" : "123-4567",
      "tel" : "03-1234-5678",
      "prefecture" : "東京都",
      "address1" : "港区サンプル1-2-3",
      "address2" : "サンプルビル",
      "person_title" : "部長",
      "person_name" : "サンプル太郎",
      "email" : "sample@example.com",
      "cc_emails" : "sample@example.com, sample2@example.com",
      "created_at" : "2015/10/31T00:00:00.000+09:00",
      "updated_at" : "2015/10/31T00:00:00.000+09:00"
    }
  ]
}
```

#### エラーレスポンス
```
HTTP/1.1 422 Unprocessable Entity

{
  "code" : "422",
  "errors" : [
    {
      "message" : "バリデーションに失敗しました。 名称を入力してください。"
    }
  ]
}
```
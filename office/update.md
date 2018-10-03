### 事業所情報の編集
```
  PATCH /api/v1/office
```

#### パラメーター
| 名称     | field              | 備考 |
| :--      | :--                | :--|
| 名前     | office[name]       | |
| 郵便番号 | office[zip]        | |
| 都道府県 | office[prefecture] | |
| 住所1    | office[address1]   | |
| 住所2    | office[address2]   | |
| 電話番号 | office[tel]        | |
| FAX番号  | office[fax]        | |

### リクエスト例
```
curl -i -H "Authorization: BEARER [ACCESS_TOKEN]" -H "Content-Type: application/json" -d '{ "office" : {"name" : "サンプル事業所2"} }' -X PATCH https://invoice.moneyforward.com/api/v1/office
```

#### レスポンス
```
HTTP/1.1 200 OK

{
  "name" : "サンプル事業所2",
  "zip" : "123-4567",
  "prefecture" : "東京都",
  "address1" : "港区サンプル1-2-3",
  "address2" : "サンプルビル",
  "tel" : "03-1234-5678",
  "fax" : "03-5678-1234"
}
```

#### エラーレスポンス
パラメータ指定方法が誤っている場合
```
HTTP/1.1 400 Bad Request

{
  "code" : "400",
  "errors" : [
    {
      "message" : "必要なパラメーターが存在しない、もしくは空です。"
    }
  ]
}
```

不正な値があった場合
```
HTTP/1.1 400 Bad Request

{
  "code" : "400",
  "errors" : [
    {
      "message" : "不正な都道府県名が渡されました。"
    }
  ]
}
```
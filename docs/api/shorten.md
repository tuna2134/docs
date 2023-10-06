# 短縮URL
ドメインがころころ変わる可能性がありますので、長い間使う方には向いておりません。

## Endpoint
`https://wakuwaka.quest`

## POST /
短縮します

**Json requests**

| name | type | description  |
| :--- | :--- | :----------  |
| url  | str  | 短縮したいURL |

**Response(applcation/json)**

| name  | type | description |
| :---  | :--- | :---------- |
| short | str  | 短縮した結果 |
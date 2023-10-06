# Webshot API
chromiumを使ってサイトのスクリーンショットを撮ってくれます。

## Endpoint
`https://webshot-api.tuna2134.jp`

## GET /
スクリーンショットを撮ります。

**Parameters**

| name   | type  | description       |
| :--:   | :--:  | :--:              |
| url    | str   | Target URL.       |
| weight | int?  | Return image size |
| height | int?  | Return image size |

**Returns**

`Bytes`で返ってきます。

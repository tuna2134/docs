# code-serverでポートフォワードするには
サービスファイルで動かしているものと想定しております。
通常だと`http://<host>:<port>/port/<target port>`でアクセスできますが、
それぞれにドメインを渡したい場合(`http://<target port>.<host>:<port>`みたいに)は以下の通りに従えばできます。

## code-serverのインストール
(省略)

## サービスファイルを変更
`/usr/lib/systemd/system/code-server@.service`を以下の通りに変更してください。

```service
[Unit]
Description=code-server
After=network.target

[Service]
Type=exec
ExecStart=/usr/bin/code-server --proxy-domain <host>
Restart=always
User=%i

[Install]
WantedBy=default.target
```

## サービスを再起動
```
sudo systemctl restart code-server@$USER
```

これでできました！

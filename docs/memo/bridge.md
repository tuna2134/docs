# ブリッジ
二つあるインターフェースを跨がせることためのインターフェースを作成する。

## ブリッジの作成
```
ip link add <bridge name>
```

## 特定のインターフェースをブリッジに参加させる
```
ip link set dev <interface name> master <bridge name>
```
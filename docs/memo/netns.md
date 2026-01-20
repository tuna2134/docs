# Linux Network Namespaceに関するメモ
通常ホストには一つのネットワークしかない。
しかし、Network Namespaceを使うことで、
分離したネットワークを作ることが可能

## 用語
- Network Namespace - 分離したネットワーク
- Init Namespace - ホストのネットワーク

## Namespaceの作成・削除
```
# 作成
ip netns add <name>

# 削除
ip netns delete <name>
```

## Init NamespaceからNetwork NamespaceにInterfaceを移動
```
ip link set netns <name> dev <interface name>
```

## 相対するInterfaceの作成
```
ip link add <host interface name> type veth peer name <namespace interface name> netns <name>
```
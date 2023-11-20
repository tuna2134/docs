# Pymark document

- pymarkとは？ - cmarkを使って、開発されたライブラリです。
- パフォーマンス　- misakaなどよりも優れています。

## Installation
```sh
git clone https://github.com/tuna2134/pymark.git
pip install ./pymark
```

以上の方法が最も成功かつ簡単なインストール方法です。(cmake, gcc必須)

## Examples
```py
import pymark


md_text = """
# Hello, World
hi
"""

print(pymark.convert(md_text))
```

## Functions

### `convert(markdown: str) -> str:`

**Argument**

- `markdown(str)` - マークダウンの内容

**Returns**

- `str` - マークダウンからhtmlに変換した内容

---
title: "git addとcommitの中身を理解する"
emoji: "🍏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git"]
published: true
---

# 目的

gitコマンドのaddとcommitの中身を図解してgitの理解を深める

# git addとcommitの中身の図

![](/images/git-add-commit/git-command-add-commit.png)

## git add

1. blobファイルを生成
    - ファイルの中身がgit管理化に置かれた状態
      - ファイルの中身をハッシュ化したものがblobのファイル名となる
    - あくまで中身だけなのでファイル名と紐付いていない
    ```sh
    $ git cat-file -p 1c59427adc4b205a270d8f810310394962e79a8b
    ファイルの中身

    $ git cat-file -t 1c59427adc4b205a270d8f810310394962e79a8b
    blob
    ```
2. blobファイルとファイル名を紐付け
    - これがステージの状態

## git commit

1. blobとファイル名の紐付けをひとまとめにするtreeオブジェクトを生成
    - ただのblobファイルをファイル名の一覧情報ファイル
      ```sh
      $ git cat-file -t 4bdc3e9b7254fb158dd4ea7dc3f14686b88c848c
      tree

      $ git cat-file -p 4bdc3e9b7254fb158dd4ea7dc3f14686b88c848c
      100644 blob 8795ee259deac221555f304d67bece194a3ce336	new_file.txt
      100644 blob 1c59427adc4b205a270d8f810310394962e79a8b	second.txt
      ```

2. treeオブジェクトからcommitオブジェクトを生成
    ```sh
    $ git cat-file -t 7b6b9615ebfb7b60dd5574dadb82b7e6ab91d246
    commit

    $ git cat-file -p 7b6b9615ebfb7b60dd5574dadb82b7e6ab91d246
    tree 4bdc3e9b7254fb158dd4ea7dc3f14686b88c848c
    parent d07d38dd95f2b7e550c20b62e6d4feec48664e89
    author hoge <hoge@example.com> 1629528534 +0900
    committer hoge <hoge@example.com> 1629528534 +0900

    add second file
    ```

3. HEADが参照しているブランチのcommit IDを2で生成されたcommitオブジェクトのIDで上書き
    ```sh
    $ cat .git/HEAD
    ref: refs/heads/main

    $ cat .git/refs/heads/main
    7b6b9615ebfb7b60dd5574dadb82b7e6ab91d246
    ```

## ファイル構成

```
.git
├── COMMIT_EDITMSG
├── HEAD
├── config
├── description
├── hooks
├── index
├── info
│   └── exclude
├── logs
│   ├── HEAD
│   └── refs
│       └── heads
│           └── main
├── objects
│   ├── 1c
│   │   └── 59427adc4b205a270d8f810310394962e79a8b
│   ├── 4b
│   │   └── dc3e9b7254fb158dd4ea7dc3f14686b88c848c
│   ├── 7b
│   │   └── 6b9615ebfb7b60dd5574dadb82b7e6ab91d246
│   ├── 87
│   │   └── 95ee259deac221555f304d67bece194a3ce336
│   ├── cd
│   │   └── 6f51860e6e43183419c5fe68a98d846a5c9e44
│   ├── d0
│   │   └── 7d38dd95f2b7e550c20b62e6d4feec48664e89
│   ├── info
│   └── pack
└── refs
    ├── heads
    │   └── main
    └── tags
```

# 参考

[Pro Git](http://git-scm.com/book/ja/v2)

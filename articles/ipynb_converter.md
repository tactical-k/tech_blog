---
title: ".ipynbと.pyを相互変換するためのツールを自作した"
emoji: "🐍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "jupyternotebook"]
published: true
---

# やったこと

仕事でJupyter Notebookを使用しております。
ただし、.ipynbファイルはGitHubで管理すると差分が見にくいという問題点がありました。
.ipynbを.pyに変換してGitHubで管理することで、差分を見やすくなります。
ここまでは、`nbconvert`コマンドを使用すれば可能です。

問題は.py形式のファイルを.ipynbに戻す方法です。
この辺りの知見を得たので、メモとして残します。

# GitHub

ライブラリとして使える状態にしておきました。
https://github.com/tactical-k/nb2py

# 今後
PyPIに登録してpipでインストールできるようにしたいです。
---
title: "FigmaMCPとFigmaPluginで出力結果を比較してみたんじゃ"
emoji: "🖼️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cursor","AI","AI駆動開発","FigmaMCP","Cursor Plugin"]
published: true
published_at: 2026-03-14 19:30
publication_name: "genai"
---
# 初めに
今回の記事は「FigmaMCP」と「FigmaPlugin」で出力結果、トークン量の違い、完成度を比較しようと思います。そんなに記載することもないので、サクッと見れるかなと思います。

# そもそもFigmaPluginとは？
詳細はこちらをご覧ください
https://zenn.dev/genai/articles/cursor-update-v2_5#cursor-marketplace-%E4%B8%8A%E3%81%AE%E3%83%97%E3%83%A9%E3%82%B0%E3%82%A4%E3%83%B3%EF%BC%88%E3%81%93%E3%82%8C%E3%81%A0%E3%81%91%E8%A6%8B%E3%82%8C%E3%81%B0%E8%89%AF%E3%81%84%E3%83%AC%E3%83%99%E3%83%AB%EF%BC%89

そんなの見てる暇がない！というざっくり言ってしますと「プラグインはスキル、サブエージェント、MCP サーバー、フック、ルールをまとめて使用できる」ものとなっています。しかも公開にはcursorチームが確認して問題ないものだけが世の中に公開されていいます。
https://cursor.com/ja/marketplace

# 今回の成果物
ごくごく普通のテーブルです。
![](/images/mcp-vs-plugin/1.png)
ね、なんの変哲もないでしょ？あくまでイメージ図ですが、こんな感じのテーブルのデータをFigmaから取得し、再現してもらおうと思います。

# FigmaMCPの場合
![](/images/mcp-vs-plugin/2.png)
なるほど、テーブルデータだけ渡したのに、必要以上に実装していますね…
じつはFigmaから取得したテーブルがあるページの全体はかなり似ているので、まぁまぁ良いような気もしますね。ただデザインはかなり異なっていたり、足りていない項目が多いので、ダメですね。
実際にこの読み取りから実装を始めると修正のほうが多いので、余計なことはせず必要な箇所だけ実装だけしてくれた方が嬉しいですね。

# FigmaPluginの場合
![](/images/mcp-vs-plugin/3.png)
あの〜完璧に再現してくれました！本当にFigmaで選択した範囲のまんまです。
もう何も言うことがなく、ありがとうって感じです。

# 最後に
いかがでしょうか？
Cursor使いならFigmaPluginの方が嬉しくないですか？やっぱりFigmaのデータをそのまんま再現してくれた点が最高ですよね。
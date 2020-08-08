---
layout: single
title:  "Jekyllのテーマを変更してGithub Pagesにブログを投稿"
date:   2020-08-09 01:30:00 +0900
categories: web
---
Jekyllのデフォルトテーマ(minima)を[minimal-mistakes](https://github.com/mmistakes/minimal-mistakes)に変更し、GitHub Pagesにブログを投稿する方法のメモです。  
※一部記憶に頼って書いているので、完全に再現できる保証はありません。

## 前提
* GitHub上にGitHub Pages用のリポジトリ(***.github.io)を作成済み
* Jekyllをインストール済

## 手順
[GitHubPagesのドキュメント](https://docs.github.com/ja/github/working-with-github-pages/adding-a-theme-to-your-github-pages-site-using-jekyll)によると、GitHub Pagesでのテーマ変更は、①[サポートされているテーマ](https://pages.github.com/themes/)か、②リモートテーマを使う方法があると書かれていますが、「①サポートされているテーマ」は、テーマの選択肢が少なく、minima以外のテーマはブログには適していませんので、「②リモートテーマ」を使うのが良いと思います。  
今回は、[minimal-mistakes](https://github.com/mmistakes/minimal-mistakes)に変更することにしました。  
基本的には[Githubのページ](https://github.com/mmistakes/minimal-mistakes)に従うだけです。
(1)Gemfileにjekyll-include-cacheプラグインを追加
```
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.6"
  gem 'jekyll-include-cache'
end
```
(2)_config.ymlのremote_themeにmmistakes/minimal-mistakesを追加(元からあったthemeは消すかコメントアウトする)  
バージョン(@4.19.3)は最新のバージョンを指定
```
remote_theme: "mmistakes/minimal-mistakes@4.19.3"
```
(3)_config.ymlのpluginsにjekyll-include-cacheを追加
```
plugins:
  - jekyll-feed
  - jekyll-include-cache
```
(4)bundleコマンドによってアップデート
```
bundle
```
以上で、minimal-mistakesのテーマをJekyllに導入することができます。ローカルでビルドする場合は、
```
bundle exec jekyll serve
```
でビルドすることができ、Github Pagesとしてビルドする場合はGithubリポジトリにpushすれば自動的にビルドされます。

## その他
* 各記事を投稿するときは、layoutをminimaのデフォルトのpostではなくsingleにする
```
layout: single
title:  "title of article"
```

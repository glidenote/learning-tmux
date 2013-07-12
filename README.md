# ターミナルマルチプレクサの話

2013年 新卒研修死霊

2013年7月12日 前田 章

## 自己紹介

 * [https://about-p-kitak.sqale.jp/users/77](https://about-p-kitak.sqale.jp/users/77)
 * [http://blog.glidenote.com/](http://blog.glidenote.com/)

## ターミナルマルチプレクサ

 * screen
 * tmux

※ screenは2年くらい使ってないので、ターミナルマルチプレクサ研修と言いつつ今日はtmuxの話しかしません

## 社内IRCのtmuxチャンネル

```
/join #tmux
```

@banyan氏がいなくなって過疎ってます…

## tmuxの無い生活

 * sshするたびにterminalのタブ開きまくり。
 * 作業がサーバが多くなるとタブも使用メモリも増えてターミナルごと死亡
 * 無線LAN(ネット回線)切れて終わり
 * `w`実行したときに大量のptsが…

## 過去に私が見たtmux(screen)使って無くてやらかした事例

 * メンテでALTER TABLEを流してる途中にターミナルが落ちて死亡
 * 十数時間かかるrsync(SCP)実行中に、回線が切れて死亡

そもそもバックグラウンドで実行とか`nohup`とかしてない時点でアウトだけど、
tmux(screen)を使ってれば回避出来た事例

## tmuxのインストール

for Mac

``` sh
brew install tmux
```

for RHEL

``` sh
yum install tmux
```

## tmuxを利用するメリット

 * 端末をデタッチ・アタッチすることができる
 * 端末上でコピー&ペーストができる
 * 別の端末から接続しても同じ作業が出来る
 * 複数の端末を起動でき、タブのように切り替えができる
 * 端末の画面分割ができる

## tmux操作の基本

 * prefixはデフォルトだと`Ctrl + b` (`Ctrl+t`に変更している人が多い気が)
 * `prefix + 他のキー` で各種操作
 * `prefix + :` でコマンド入力モードに入る
 * 各種キーや設定は`~/.tmux.conf`でカスタム可能
 * ショートカットキーの表示 `prefix + ?`

## windowとpane

![](https://raw.github.com/glidenote/learning-tmux/master/images/tmux-image0.png)

screenにはpaneという概念がないので、screenからの移行してくると戸惑う。

## デタッチ、アタッチ

 * 作業を中断する場合はデタッチ。デタッチした時点の状態が保存されている
 * 作業を再開する場合はアタッチ。デタッチ時の状態が復元される

## コピーモード

 * window間でのコピペが可能
 * コピーモード開始 `prefix + [`
 * コピー始点選択 `space`
 * コピー終点 `Enter`
 * 貼り付け `prefix + ]`
 * 矩形選択なら`space`の後に`v`

## tmuxの使いどころ1

``` txt
                      +--------------+
                      |   server2    |
                      +--------------+
                        ^
                        |
                        |
+---------------+     +--------------+     +---------+
| Local Machine | --> | manage(tmux) | --> | server3 |
+---------------+     +--------------+     +---------+
                        |
                        |
                        v
                      +--------------+
                      |   server1    |
                      +--------------+
```

 * ssh先の`manage`サーバでtmuxを起動して、`server{1,2,3}`にssh接続する。`Local Machine`と`manage`間の通信が切れても大丈夫
 * `manage`と`server{1,2,3}`は同じデータセンター内で基本的に通信が切れないことが前提

※ graph-easyを使うと上のような図は簡単に作れるので覚えておきましょう

## tmuxの使いどころ2

tmux上でpuppetファイルの編集作業を実際にやってみる

下記のwindowで作業

 * zsh
 * vim
 * puppet-server(manage)
 * puppet-client(server1)
 * puppet-client(server2)


## window操作

 * windowのリネーム `prefix + ,`
 * 新しいwindowの作成 `prefix + c`
 * window一覧の表示 `:list-window`
 * windowの移動 `:move-window -t 6`
 * windowを入れ替える `:swap-window -t 1`

## pane操作

 * pane入れ替え `prefix + {` or `prefix + }`
 * pane番号の表示 `prefix + q`
 * レイアウトの変更 `prefix + space`
 * サイズ変更 `+ - < >`
 * pane一覧の表示 `:list-pane`
 * paneを:1ウィンドウから移動してくる `:join-pane -s :1`
 * paneを:1ウィンドウに移す `:join-pane -dt :1`
 * activeなpaneがwindowの番号のwindowに移動 `:join-pane -t :動かしたい先のwindowの番号`
 * windowの番号のpaneがactiveなpaneに移動 `:join-pane -s :もって来たいpaneがあるwindowの番号`
 * 特定windowの特定paneに移動する `:join-pane -s :持ってきたいpaneがあるwindowの番号.pane番号`
 * paneの破棄 `prefix + x`

## 応用編

## .tmux.confのカスタム1

``` conf
# reload .tmux.conf
unbind r
bind   r source-file ~/.tmux.conf \; display-message 'source-file ~/.tmux.conf'
```

を設定しておくと`prefix + r `で設定再読込。

## .tmux.confのカスタム2

下記のように短縮して記載出来るので、活用しましょう。

``` conf
set-option        => set
set-window-option => setw
bind-key          => bind
unbind-key        => unbind
```

`man tmux`に`alias`が書いてあります

## display messageを使って通知

``` sh
sleep 5 ;tmux display-message "hogemoge is done."
```

 * `~/.tmux.conf`に`set-option -g display-time 10000`を設定
 * 何かキーを押すと消えるので使い勝手はイマイチ
 * リモートサーバだと使えない

## ssh(mosh)するごとにwindowを自動生成する

 * [SSH接続時にホスト名を付けたtmuxウィンドウを開く方法 - けめの日記](http://keme.hatenablog.com/entry/2012/10/20/011327)
 * [tmux上でmosh接続したときに新しいウィンドウを生成する - Glide Note - グライドノート](http://blog.glidenote.com/blog/2012/04/11/mosh-with-tmux/)

## 他のwindowのpaneを持ってくる

 * [tmuxのペイン切り替えをscreenみたくする(ターミナルマルチプレクサ Advent Calendar 2011 23日目) - kozo2のはてなダイアリー](http://d.hatena.ne.jp/kozo2/20111223/1324667710)

## pipe-paneで作業ログを取る

 * [tmuxのpipe-paneを利用してリモートサーバでの作業ログをローカルに記録する - Glide Note - グライドノート](http://blog.glidenote.com/blog/2013/02/04/tmux-pipe-pane-logging/)

## tmuxのコピーモードとMacのクリップボードを連携

 * [X環境のクリップボードやOS Xのペーストボードとtmuxのバッファを連携する方法 - Dマイナー志向](http://d.hatena.ne.jp/tmatsuu/20111220/1324399472)
 * iTerm2にそれ系の機能があるぽい(私は使ってないです)

## tmuxあるある

 * 他の端末からアタッチ`tmux a`したときに画面がおかしくなる => `tmux a -d`を使いましょう
 * ウィンドウサイズの異なる端末から接続しても残念な感じにならない!!
 * [tmux で画面の外に |----------------- ってでまくるときの対策 - tokuhirom's blog.](http://blog.64p.org/entry/2013/04/26/191706)

## まとめ

## tmuxの心得

 1. リモートサーバに接続したら、すぐに起動
 1. tmux使ってても事故ることあるので、window名の自動設定とか利用する
 1. tmuxを使いこなして凡ミスを防ごう
 1. 賢人のdotfiles(`.tmux.conf`)を参考にしよう

## 上記に書いた以外の参考死霊


 * [時代はGNU screenからtmuxへ - Dマイナー志向](http://d.hatena.ne.jp/tmatsuu/20090709/1247150771)
 * [ターミナルマルチプレクサ Advent Calendar 2011 : ATND](http://atnd.org/events/22320) (@banyan氏と私も書いてます)


\clearpage

# はじめに
現代では脳は電気で動いている、と信じられています。
しかし、どのような挙動なのかはまだまだ分かっていません。
だから、貴方は研究をしたくなります。(それは火を見るより明らかです)
しかし、脳の解析は難しく、技術的な入門書、特に和書に乏しい現状があります。
だから同人誌を書くことにしました。
本書では脳磁図、脳波、MRI解析を「体で覚える」べく実践していきます。
さぁ、MNE/python、freesurferの世界で良い生活を送りましょう！

## 本書の目的・限界・対象者・使い方
MNE/python[^about-mne][^about-mne2]やfreesurfer[^about-freesurfer]を用いて脳内の電源推定…特にソースベース解析を行うための
解析環境の構築と解析の基礎を概説します。可能な限り効率的な解析環境を構築し、楽をします。
僕が個人的に考えている事もちょくちょく書きます。
僕はelekta社のMEGを使っているのでelekta前提で書きます。

[^about-mne]:Gramfort, M. Luessi, E. Larson, D. Engemann, D. Strohmeier, C. Brodbeck, R. Goj, M. Jas, T. Brooks, L. Parkkonen, M. Hämäläinen, MEG and EEG data analysis with MNE-Python, Frontiers in Neuroscience, Volume 7, 2013, ISSN 1662-453X
[^about-mne2]:Gramfort, M. Luessi, E. Larson, D. Engemann, D. Strohmeier, C. Brodbeck, L. Parkkonen, M. Hämäläinen, MNE software for processing MEG and EEG data, NeuroImage, Volume 86, 1 February 2014, Pages 446-460, ISSN 1053-8119
[^about-freesurfer]:https://surfer.nmr.mgh.harvard.edu/fswiki/FreeSurferWiki

本書の限界は僕のスキル不足と、これが同人誌であること、
MNE自体の進化のスピードが光の速さであることです。
不確実なものとして、疑って読んでいただければ幸いです。
この同人誌は不完全なため、日々更新しています。

本書の対象者は以下のとおりです

- 脳波/脳磁図計を使って研究をしたい初心者
- 脳磁図計を使って研究しているけれど、コーディングが苦手な中級者
- 頭部MRI研究でfreesurferを使いたい初心者

また、前提条件としてターミナルやプログラミングを怖がらないことがあります。
(プログラミング未経験者の質問にも出来るだけ答えたいと思います)

脳研究の経験者は[MNE/pythonとは](#MNEpythonとは)から読んでいけばいいです。
MNE/freesurfer経験者なら[OSの準備](#OSの準備)から読めばいいです。
コンピュータは自転車みたいなもので、基本は体で覚えていくしかないと思っています。
分からなければググることが大事です。qiita[^qiita]等で検索するのも良いでしょう。

## センサーベース/ソースベース解析とはなんぞや
脳の中の電気信号を調べる方法としては脳波や脳磁図[^meg]が有名です。
脳波や脳磁図のセンサーで捉えた信号を直接解析する方法を
センサーレベルの解析と言います。これは伝統的なやり方であり、
今でも多くの論文がこの方法で出ている確実な方法です。

しかし、脳波や脳磁図は頭蓋骨を外して直接電極をつけないと
発生源(僕達はソースと呼びます)での電気活動はわかりません[^doubutsu]。
普段計測している脳波・脳磁図は所詮は「漏れでた信号」に過ぎないのです。
では、一体どうすれば脳内の電気信号を非侵襲的に観察できるのでしょうか？
方法は残念ながら**ない**[^nai]のですが、推定する方法ならあります。
その中の一つの方法として、脳磁図とMRIを組み合わせ、
MNEというpythonパッケージを使って自ら解析用スクリプトを実装する方法があります。
ソースベース解析というのはあくまで推定であり、
先進的である一方でまだまだ確実性には劣るやり方との指摘もあります。

ちなみに、脳波のソースベース解析もあるにはあるのですが、
脳波は電流であるため磁力と違って拡散しやすい性質があります。
実際、脳波でのソースベース解析とセンサーベース解析の結果が
不一致であったという研究が発表されています。[^huicchi]

## スラングや用語の説明とunix系のお約束
本書では下記の言葉を使っています。伝統的なスラングを含みます。
適宜読み替えていってください。それ以外にも色々スラングあるかもです…。
何故スラングをそのまま書いているかって？
お前は同人誌にまで正しい日本語を求めるのですか？そういう人は回れ右。

- hoge:貴方の環境に応じて読み替えてください、という意味のスラング
 fuga,piyoも同じ意味です。ちなみにこれは**日本語**です。
 英語が好きな方はfooとかbarとかになりますね。
- 叩く:(コマンドをターミナルから)実行するという意味の他動詞
- 回す、走らせる:重い処理を実行するという意味の他動詞
- ターミナル:いわゆる「黒い画面」のこと。Macならユーティリティフォルダにある。
- .bash_profile:ホームディレクトリにある隠し設定ファイルです。
 環境によって.bashrcだったりしますし、両方あることもあります。
 貴方の環境でどちらが動いているか(両方のこともある)確認して設定してください。
- 実装:プログラミングのことです。プログラムを書くことです。

本書で「インストールにはこうします」とか言ってコマンドを示した場合は
文脈上特に何もない場合、ターミナルでそれを叩いてくださいという意味です。
pythonの文脈になったらpythonです。この辺りは見慣れれば判別できます。


## それぞれの研究に必要な環境・特色のまとめ
本書ではまず環境を構築しますので、色々インストールが必要です。
必要物品についてまとめると下記です。

- 脳波センサーレベル研究
 python(本書ではanaconda使用)、MNE/python
 安価で普及していますが、まだ多くの謎が眠っている分野です。
 脳回や深部の信号に強いですが、脳脊髄液や頭蓋骨を伝わって行くうちに
 信号が拡散してしまうため、空間分解能が低いです。
- MEGセンサーレベル研究
 python(本書ではanaconda使用)、MNE/python
 ノイズに弱く、脳の深部に弱く、莫大な資金が必要な希少な機器です。
 それさえクリアできれば処理の重い脳波みたいなものです。
 脳波と違って脳溝をよく見れます。また、拡散しにくいので空間分解能は高めです。
- MRI研究
 freesurfer、mricrogl(mricron)
 脳内の水分子に磁場を与えて画像化する凄い機械です。
 莫大な資金が必要ですが、それなりに普及しています。
 血流を見ることが出来るので「脳機能」も見れます。
 ネタが尽きようとも、新たな理論を持ち出してくる根性の分野です。
 ※グラフ理論で解析する場合はpython必要
- MEG/EEG+MRIソースレベル研究
 MEGとMRIを組み合わせて脳の電気活動を見るやつです。
 本書の本題です。MRIの空間分解能と脳磁図の時間分解能を備えた
 まさに **†最強の解析†** …のはずなんですが、どうなんでしょうね？
 実際はそこまで空間分解能は高くないっす。モワッとしてます。
 ※膨大な計算量が必要です。

[^qiita]:日本のプログラマ用のSNSの一つです。

[^doubutsu]: 動物実験では脳に電極刺す実験はされていますが、人に刺すと警察に捕まります。

[^meg]: 脳波は電気信号を捉えますが、脳磁図は磁場を捉えます。電気と違って骨を貫通しやすく拡散しにくいので空間分解能に優れますが、ノイズに弱いです。値段も高いです。http://www.elekta.co.jp/products/functionalmapping.html

[^nai]: 他に脳の活動を調べる方法として磁力を照射するfMRIや赤外線を照射するNIRSなどがあります。fMRIは電気信号ってわけでも無さそうです。NIRSは赤外線で脳血流を捉えるのですが、頭皮の血流をいっぱい拾ってしまうので大変です。

[^huicchi]:http://biorxiv.org/content/early/2017/03/29/121764

## MNE/pythonとは
脳磁図を解析するためのpython[^python]用numpy,scipyベースのパッケージです。
自由度が非常に高いです。(引き換えに難易度が高いです。)
ソース推定をするためのパッケージなのですが、
Wavelet変換とかICAとかPermutationとか、その他あらゆる事が出来ます。
出来るのですが…使いこなすためには生理学、数学、工学の知識が必要です。
ちなみに元来脳磁図用なのですが、脳波を解析することも出来ます。
C言語で実装されたMNE/Cというのもありますが、古いバージョンと考えていいです。[^old]
最近はMNEpythonに機能を移しています、移行がまだ完全ではないところがあるなら
必要かもしれません。両方共フリーウェアですが、MNE-Cは登録が必要です。
開発は活発で、最近新バージョンは[MNE/python 0.16.1](http://martinos.org/mne/stable/index.html)です。
freesurferは[6.0](https://surfer.nmr.mgh.harvard.edu/)が、pythonはpython3.7が最新です。
導入と紹介を書いていこうと思います。
最近MNEがアップデートされて、python3シリーズが使えるようになりました！
pythonはpython3.6を使っていくことになります。
python2はマジでオワコンなので特別な理由がない限り使わないように。いいね？

![wavelet変換の出力例](img/wavelet.png){width=14cm}

[^old]: 良いところもあるんですよ？
[^python]:コンピュータ言語の一つ。速度を犠牲にして、読み書きやすさを追求した言語。科学計算の世界では現時点では広く普及しています。MATLABと似ていますが、pythonは無料でオブジェクト指向の汎用言語なので、応用範囲がWebサーバーとか機械の制御にまで及び、習得して損をすることはまずないでしょう。

## freesurferとは
頭部MRIを解析する為のソフトです。自動で皮質の厚さやボリュームを測れるだけでなく最近は
fMRIでコネクティビティの算出が出来るようになるなど、かなり賢いです。特に厚さに強いです。
反面、激重な上にサイズが大きくターミナル使う必要があります。
Unix系OSじゃないと動きません。
その上、違うCPU使ったら結果が変わる仕様があり、正しく扱わないとジャジャ馬と化します。
最近頭部MRI研究で勢力を伸ばしつつあり、最早スタンダードの一つだそうです。フリーウェアです。

\newpage
## それぞれのソフトの関係性〜それofficeに例えるとどうなの？〜
実は、以前人に教えようとした時、
いきなりMNEpythonと言われても初心者にはよくわからないと言われました。
unix系もコンピュータ言語も触ったことない人には例え話のほうが良いかもしれないので、
初心者のために、登場するソフトの名前を例え話で話してみます。
凄く乱暴な例えではあります。

| MNE                   | 役割                           | オフィスに例えると？                   |
|-----------------------|--------------------------------|----------------------------------------|
| anaconda,pip,homebrew | ソフトをインストールするソフト | app store, google play, 人事部         |
| spyder,jupyter        | 実際に色々書いたりするソフト   | word, excel, 筆記用具                  |
| python                | 言語                           | 日本語, 命令書の書式, 社内文書         |
| MNE                   | 言語で動く命令セット           | excelの関数, 社内文書に従って動く部下  |
| mricron/mricrogl      | 変換・表示用ソフト             | 画像変換ソフト, 通訳                   |
| freefurfer/freeview   | MRI画像処理ソフト              | Photoshop, 絵の具, MNEと違う部署の部下 |

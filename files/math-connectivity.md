
# コネクティビティ各論
コネクティビティについては色んなmethodがあります。
なので、スペクトルとは章を分けてみました。
ここではコネクティビティそれぞれのmethodについて
僕の考えを述べます。間違ってたらごめんね。

## 王道のPLVとCoherenceとその問題点

さて、フーリエ変換のところでコネクティビティについてちらりと書きました。
位相の差をとっていけばPLVという「どのくらい波が関連しているか」という
指標になると書きました。式で表すと

$$PLV = |\overline{\frac{Sxy}{|Sxy|}}|$$

という感じです。
この指標は正しいです。が、大きな欠点があります。
脳波にしろ、脳磁図にしろ、脳内に電極をブチ込むやり方ではなく、
漏れて拡散してきた物を捉えることになります。
ということは、何らかの大きな震源が近くにある場合、
影響を受けて似たような波が出た全てのチャンネルは
コネクティビティが「ある」と間違った結果が出てきてしまうのです。
図に沿って言うと、2つの青い点(センサー)で、一つの赤い波を同時に測定すると、
繋がっていると勘違いするのです。常識的におかしい。

![PLVでコネクティビティを計算する場合、この図の青い点が繋がっていると言うことになる。](./img/plv.png){width=14cm}

さて、PLVについて語ってきましたが、Coherenceという計算の方法もあります。
以下のように計算します。これもまた、違和感のない計算方法ですが、
PLVと同様に拡散やノイズの影響が大きいです。

$$Coherence = \frac{\overline{|Sxy|}}{\sqrt{\overline{|Sxx|} * \overline{|Syy|}}}$$

ちなみに、Coherenceに位相情報を残したものがCoherencyです。
$$Coherence = \frac{\overline{Sxy}}{\sqrt{\overline{|Sxx|} * \overline{|Syy|}}}$$
こいつの使いみちは僕はあまり知らないです…

## PLVやCoherenceの欠点の克服
いくつか方法があります。
大きく分けて2つ、またはその組み合わせです。

- 拡散する前の電流を推定する
- 拡散しても大丈夫な計算方法を採る

### 電流源推定
センサーベースで拡散するならソースベースにしちゃえばいいじゃない。
というと、ソースベース解析が出来ない人からするとマリー・アントワネット的に
聞こえるかも知れませんが、本書はソースベース解析を主眼としている本なので…
MNEでもsLORETAでもdSPMでも使ってやればいいんじゃないですかね。

もう一つの方法がCurrentSourceDensityという方法です。
こっちはソースベース解析ほどバリバリに推定するわけではない感じですが、
PLVの弱点を補う方法ですね。
残念ながらMNEpythonには未実装です。

### PLVの発展系
MNEpythonに実装されている有名所として、PLIとWPLIを紹介します。

#### PLI
PhaseLagIndexという指標があります。
こいつは上記の拡散をキャンセルしてしまう方法で、
MNEではspectral_connectivity関数に実装されています。
式は下記

$$PLI = \overline{|sign(Im(Sxy))|}$$

ここで、signは内容が正なら1、0なら0、負ならｰ1を返す関数。
Imは虚軸だけを返す関数です。
位相が常にxより進んでいたり、遅れている奴だけ加算していき、
位相のズレが0付近をウロウロしてるやつやバラバラなやつは消すメソッドですね。
確かにこれならばさっき挙げた偽のコネクティビティは出ないでしょう。
僕は初見「マジカヨ…」って思いました。
これでConnectivityをPlotしたら、隣同士ばかり繋がるような図じゃなくなります。
つまり、成功しているということですかね。

#### WPLI
上記で、正なら1、負ならｰ1というのについてマジカヨ感が漂うのですが、
そこをもう少しスムーズにしたのがWeightedPhaseLagIndexです。
これは位相のズレが$\pi/2$に近ければ近いほど大きな値を、
$-\pi/2$に近ければ近いほど小さな値を入れ込みます。
そうすることによって、ノイズに強くなった…らしいです。
そりゃそうですよね。PLIは一寸ノイズがあったら1か-1に振り切れますもの。
式は以下。

$$WPLI = \frac{|\overline{Sxy}|}{\overline{|Sxy|}}$$

見にくい記法ですみません…


### Coherenceの発展系
さて、PLI系は同一の電流源からのラグをとっていたわけですが、
同様のやり方がImaginaryCoherenceです。

これはCoherence虚軸のみを加算平均したものです。
虚軸を加算平均するということは、位相が$\pi/2$ずれたら最大になりますね。
式は下記のような感じです。

$$Coherence = \frac{\overline{Im(Sxy)}}{\sqrt{\overline{|Sxx|} * \overline{|Syy|}}}$$

### で、こういうのってロバストなの？
分からない…僕には何もわからないんです…

## グラフ理論
さらなる解析として、グラフ理論があります。
これは「互いにどんな風に繋がっているかな？」というのを
考えていく理論です。色々なやり方があります。
例えば、一筆書きでどんな風に繋ぐか、ループを作らずにどんな風に繋ぐかとか、
そういうノリのやつです。後術します。
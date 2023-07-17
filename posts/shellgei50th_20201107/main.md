---       
Keywords: シェル芸勉強会,bash,paste,factor,shuf,bashcms2,
articleTitle: 第50回シェル芸勉強会      
Copyright: (C) 2020 sickleaf       
---       

<br>
記念すべき第50回。今回の問題は、苦手としている分野（文字コードや画像を扱うもの）が無く、個人的に日々面している課題のヒントになるとても有意義な問題セットでした。       
<br>
（苦手分野が出る問題セットも勿論有意義です！）       
<br>
問題全８問は後ほど公開されるため、特に面白かった問題や回答について触れてみます。       
<br>
<br>
## 第50回Q4      
<br>
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">## Q4<br><br>　1から1億までの数字をシャッフルして2列にしたデータ（ファイル名は`a`に<br>しましょう）を作ってください。速いワンライナーを考えてください。例を示<br>します。<br><br>```<br>$ head -n 5 a<br>30704298 56976301<br>61041738 68147433<br>99052527 91351967<br>63294008 15458140<br>3840917 37301114<br>```<a href="https://twitter.com/hashtag/%E3%82%B7%E3%82%A7%E3%83%AB%E8%8A%B8?src=hash&amp;ref_src=twsrc%5Etfw">#シェル芸</a></p>&mdash; 上田 隆一 (@ryuichiueda) <a href="https://twitter.com/ryuichiueda/status/1324941729514991616?ref_src=twsrc%5Etfw">November 7, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>       
<br>
1億が難しければ1万くらいにスケールを落としてまずやってみても良い。       
<br>
まず最初の問題はすぐ方針が立つかどうか？1万で出来なかったにせよ、どの辺のコマンドを使うか当たりが付けてmanを読んで…としているうちに私は時間が終わってしまいました。       
<br>
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">あーやっぱshuf | xargs -nダメなのか <a href="https://twitter.com/hashtag/%E3%82%B7%E3%82%A7%E3%83%AB%E8%8A%B8?src=hash&amp;ref_src=twsrc%5Etfw">#シェル芸</a></p>&mdash; 病葉(Yu Kidawara) (@sickleaf3) <a href="https://twitter.com/sickleaf3/status/1324942299801939968?ref_src=twsrc%5Etfw">November 7, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>        
<br>
<br>
<br>
連番生成→シャッフル→2個ずつで折り返す、という方針は単純ですが、書き方一つで大違い。       
<br>
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">エグい性能差 <a href="https://t.co/IC1a38I4ul">pic.twitter.com/IC1a38I4ul</a></p>&mdash; 病葉(Yu Kidawara) (@sickleaf3) <a href="https://twitter.com/sickleaf3/status/1325083648563949568?ref_src=twsrc%5Etfw">November 7, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>       
<br>
shuf | xargs -n2はダメです。。       
<br>
<br>
paste 【列数の数だけ-を入れる】　というのはマジ盲点。　そもそもpasteの使い方をちゃんと分かってなかった。       
あと地味にseqの後の数字が指数表記出来ることを知った。便利。       
<br>
<br>
シェル芸勉強会のTLは大変恐ろしいので、マイナーのオプションや別解が出てきます。       
<br>
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">(time shuf -i 1-1000000|paste -d\  - -&gt;a) |&amp; cat<a href="https://twitter.com/hashtag/%E3%82%B7%E3%82%A7%E3%83%AB%E8%8A%B8?src=hash&amp;ref_src=twsrc%5Etfw">#シェル芸</a></p>&mdash; hiro (@hiro_poke91) <a href="https://twitter.com/hiro_poke91/status/1324948983463071744?ref_src=twsrc%5Etfw">November 7, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>        
shufの-iオプションで、min-maxを指定するとseq min maxと同じ扱いになるという…
<br>
## 第50回Q6      
<br>
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">## Q6<br><br>　`a`について、両方の数字が素数の行を抽出してファイルに保存してくださ&gt;い。<a href="https://twitter.com/hashtag/%E3%82%B7%E3%82%A7%E3%83%AB%E8%8A%B8?src=hash&amp;ref_src=twsrc%5Etfw">#シェル芸</a></p>&mdash; 上田 隆一 (@ryuichiueda) <a href="https://twitter.com/ryuichiueda/status/1324953706912116737?ref_src=twsrc%5Etfw">November 7, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>       
<br>
aというのは、Q4で作成した数字が2列ずつ大量に並んだファイルのこと。       
<br>
素数を扱う時のお決まりのコマンドfactor。例えば24なら`24: 2 2 2 3`と表示されるので、数字が1列のときはfactorを通して出力を処理するのだが、さて2列並んでるからどうするのか…？       
<br>
5000万行・両方の列の数字に対してfactorを掛けて、そこから両方素数のものだけに絞る…となかなか手ごわい処理。       
<br>
まず知らなかったのは、factorは複数数字が並んでいても処理出来るということ。       
<br>
<blockquote class="twitter-tweet"><p lang="und" dir="ltr">9999: 3 3 11 101<br>10000: 2 2 2 2 5 5 5 5<br>10001: 73 137 <a href="https://t.co/qqJvNBVRmH">https://t.co/qqJvNBVRmH</a></p>&mdash; シェル芸bot (@minyoruminyon) <a href="https://twitter.com/minyoruminyon/status/1324955806941372416?ref_src=twsrc%5Etfw">November 7, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
<br>
後は素数判定で、気づいてみれば何てこと無いんだけど思いつけるかどうか。       
<br>
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">cat a | factor | paste - - | awk &#39;NF==4{print $2, $4}&#39; &gt; primes.txt<a href="https://twitter.com/hashtag/%E3%82%B7%E3%82%A7%E3%83%AB%E8%8A%B8?src=hash&amp;ref_src=twsrc%5Etfw">#シェル芸</a></p>&mdash; eban (@eban) <a href="https://twitter.com/eban/status/1324952980982882304?ref_src=twsrc%5Etfw">November 7, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>        
<br>
素数かどうかで表示される列数が変わるので、awkで絞れるという仕組み。（cat aの部分はseq , shuf, pasteを使うQ4の部分を使えばワンライナーになる）。　思いつけば簡単なのだが…。。       
<br>
<br>
更に、素数判定するので偶数が出てくる列は除外出来ます。そうするともう少し性能が上がる。       
<br>
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">AND条件への変更は特に覿面。ORのときより20秒程度の短縮を実現。 <a href="https://twitter.com/hashtag/%E3%82%B7%E3%82%A7%E3%83%AB%E8%8A%B8?src=hash&amp;ref_src=twsrc%5Etfw">#シェル芸</a><br>$ time awk &#39;($1%2 &amp;&amp; $2%2)&#39; a | factor | paste - - | awk &#39;(NF==4){print $2,$4}&#39; &gt; c<br><br>real0m39.376s<br>user1m9.255s<br>sys0m8.089s</p>&mdash; Hisatoshi Onishi (@hisa0211) <a href="https://twitter.com/hisa0211/status/1324958659546284033?ref_src=twsrc%5Etfw">November 7, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>        
<br>
<br>
Q7,Q8はここで纏めるより1記事として独立して書けそうな良問なので、出来れば振り返りを行いたい。       
<br>
問題のメインテーマが高速化で複数のアプローチがあるので、高速化の色んな知見が溜まる回でした。       
<br>
<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">今日の学び<br><br>topでパイプのボトルネックと並列度を見る<br>前処理大事<br>peeで並列化<br>teipやばい<br>numsumで足す<br>paste速い<br>shuf -iで範囲指定<br>sortは直読みで並列化で速い<br>tacは直読みで速い<br>sort -mでソート済みをマージ<br>xargs -P0で全力並列<br>moreutils便利<br><br>ありがとうございましたー<a href="https://twitter.com/hashtag/%E3%82%B7%E3%82%A7%E3%83%AB%E8%8A%B8?src=hash&amp;ref_src=twsrc%5Etfw">#シェル芸</a></p>&mdash; 近藤健夫 (@takeoverjp) <a href="https://twitter.com/takeoverjp/status/1324970584099364864?ref_src=twsrc%5Etfw">November 7, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>        
<br>
振り返り大事。       
<br>
因みにLTやろうかなと悩んでおり、出来るか判断するためにまずは配信環境整えてその後にスライド作ろうとしたのだけど、YouTubeのライブ配信の有効設定をしてから配信出来るまで24時間掛かることを知った途端諦めてしまった。
<br>
前日・当日よりもう少し考えて動くべきだった。。
<br>
<br>
## 最後に      
<br>
このブログはbashcms2上で動いており、記事ドラフトはスプレッドシートで書いております。       
 

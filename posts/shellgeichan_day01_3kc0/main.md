---
articleTitle: 世界のナベアツFizzBuzz（○○するシェル芸ちゃん_1日目）
Keywords: ○○するシェル芸ちゃん, bash, awk, fizzbuzz, seq,シェル芸
createdTime: 2021-12-01 23:59:56
Copyright: (C) 2021 sickleaf
---
<br>

[○○するシェル芸ちゃん](../../?page=advent_shellgeichan)アドベントカレンダーの1日目です。
<br>

**FizzBuzz**という有名な問題があります。

<br>
**数字を列挙し、「3の倍数はFizz」「5の倍数はBuzz」を出力する**基本的な処理が書けるかを問うプログラミングの課題です。


一方で芸能で"3の倍数"というと、<font style="color:red;">白目で口を歪ませてアホになる例のアレ</font>を連想する人が一定数居るはずです。

アレには"3のつく数"という条件があるので、FizzBuzz問題に取り込んでみました。


そしてこのアドベントカレンダーのメインはシェル芸なので、シェル芸での回答を目指していきます。

#### 問題

```
問. 1から100までの数字を列挙して、
「3の倍数の場合はAho」「5の倍数の場合はBoke」「3がつく数字の場合はKasu」と表示して下さい。
例：
12: Aho
13: Kasu
14: 
15: AhoBoke
```


シェル芸が出来る人はここで回答を、何が何やら分からない人はそのまま読み進めて下さい。


### 回答方針

やることは、「数字の列挙」「出力処理」「条件の判定」の３つです。


ここまで「何のこと？」という人がまあまあ居るかと思いますが、もう少しだけご辛抱を。

流石に**わからん人ほっときますよ、義務教育ちゃうねんから**とは言いません。


#### 数字の列挙

1から100までの列挙には`seq`コマンドを使います。

```
$ seq 100
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
```

[このサイト](https://websh.jiro4989.com/)を開き、inputに`seq 100`を入力して、`Run(Ctrl+Enter)`を押してみると、stdoutに同じ結果が表示されるはずです。

<br>

この表示は、【Unix系コマンドが備わったシェル上で実行する】と出てきます。実はMacOSを使っている人はターミナルを開くと出来たりするのですが…。


このようにコマンドの複雑に組み合わせ、欲しい結果を得る処理をワンライナー（**oneliner**）／シェル芸と呼ぶことがあります。シェル芸の厳密な定義は以下の通り。

>マウスも使わず、ソースコードも残さず、GUIツールを立ち上げる間もなく、あらゆる調査・計算・テキスト処理をCLI端末へのコマンド入力一撃で終わらすこと。あるいはそのときのコマンド入力のこと。
>
>（シェル芸の定義バージョン1.1）

<br>

ここまで説明すれば今日は終わりです。

後は淡々と答えを作っていきます。


#### 出力処理

`seq`の出力に続けて`awk`を使います。こんな感じ。

```
# 数字の後に":"を出す
$ seq 100 | awk '{print $0":"}'
（略）
```


#### 条件の判定

指定された条件によって、出力を書いていきます。

##### 3の倍数、5の倍数

**数の余りが0かどうか**で判定します。（`if($0%3==0)`、`if($0%5==0)`）

##### 3がつく数字

**その行が3を含んでいるか**で判定します。（`if($0 ~ /3/)`）


### 回答例

`seq 100 | awk '{printf "%d:", $0; if($0%3==0){printf "Aho"} if($0%5==0){printf "Boke"} if($0 ~ /3/){printf "Kasu"};
 printf "\n"}'`

```
$ seq 100 | awk '{printf "%d:", $0; if($0%3==0){printf "Aho"} if($0%5==0){printf "Boke"} if($0 ~ /3/){printf "Kasu"}; printf "\n"}'
1:
2:
3:AhoKasu
4:
5:Boke
6:Aho
7:
8:
9:Aho
10:Boke
11:
12:Aho
13:Kasu
14:
15:AhoBoke
16:
17:
18:Aho
19:
20:Boke
21:Aho
22:
23:Kasu
24:Aho
25:Boke
26:
27:Aho
28:
29:
30:AhoBokeKasu
31:Kasu
32:Kasu
33:AhoKasu
34:Kasu
35:BokeKasu
36:AhoKasu
37:Kasu
38:Kasu
39:AhoKasu
40:Boke
41:
42:Aho
43:Kasu
44:
45:AhoBoke
46:
47:
48:Aho
49:
50:Boke
51:Aho
52:
53:Kasu
54:Aho
55:Boke
56:
57:Aho
58:
59:
60:AhoBoke
61:
62:
63:AhoKasu
64:
65:Boke
66:Aho
67:
68:
69:Aho
70:Boke
71:
72:Aho
73:Kasu
74:
75:AhoBoke
76:
77:
78:Aho
79:
80:Boke
81:Aho
82:
83:Kasu
84:Aho
85:Boke
86:
87:Aho
88:
89:
90:AhoBoke
91:
92:
93:AhoKasu
94:
95:Boke
96:Aho
97:
98:
99:Aho
100:Boke
```

「何やってるんだ？」と思ったあなたは、きっとまともな人間です。


<blockquote class="twitter-tweet"><p lang="eu" dir="ltr">20:Boke<br>21:Aho<br>22:<br>23:Kasu<br>24:Aho<br>25:Boke<br>26:<br>27:Aho<br>28:<br>29:<br>30:AhoBokeKasu<br>31:Kasu<br>32:Kasu<br>33:AhoKasu<br>34:Kasu <a href="https://t.co/jNheQlGzf7">https://t.co/jNheQlGzf7</a></p>&mdash; シェル芸bot (@minyoruminyon) <a href="https://twitter.com/minyoruminyon/status/1466062581400064006?ref_src=twsrc%5Etfw">December 1, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


### ○○する芸能ちゃんブロック

ここまでシェル芸を知っている人向けに淡々と回答を書いていきましたが、ここからは○○する芸能ちゃんブロックです。


<font style="color:red;">自分ですら先が見えていない一人アドベントカレンダーをブチち上げる原動力は何か</font>、その一端をシェル芸人の皆さんに日々感じていただければと思います。

<br>
[アドベントカレンダーのリンク](../../?page=advent_shellgeichan)に書いている通り、このアドベントカレンダーは[○○する芸能ちゃん](https://www.youtube.com/channel/UCWQQMejT4ve8IFeKTWV6AsA)というYouTubeチャンネルに着想を得て名付けました。

要は**贔屓しているYouTubeチャンネル**ということです。

**芸能バカにはたまらない**動画がいくつかあり、私の思う順番に見て欲しいというエゴによって先の長いアドベントカレンダーを書き始めてしまいました。

ということで1日目にオススメするのはこちらです。

### 今日のオススメ動画

○○する芸能ちゃんを代表する動画は他にあるのですが、**導入として見てもらう動画はこの動画がベストだろう**と私は思っております。

ラスト1,2分近くで、**初見にはインパクトの強い映像効果がありますのでご注意下さい。**

<div class="youtube">
<iframe width="560" height="315" src="https://www.youtube.com/embed/IJ_aijyUZrE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

**１．ズバ抜けた編集の凄さ**

**２．素の喋りの上手さ**

**３．芸能マニア度の高さ**

この３点を感じて頂ければ。

<style>
.youtube {
  position: relative;
  width: 100%;
  padding-top: 56.25%;
}
.youtube iframe {
  position: absolute;
  top: 0;
  right: 0;
  width: 100% !important;
  height: 100% !important;
}
</style>


<!--


## 書籍近影

### 別ページへのリンク
[memolink](../../?page=memolink)

### 複数行引用
<blockquote></blockquote>

#### CSS埋め込み

<style>
.col-md-9 > iframe {
	width:100% !important;
	height:500px !important;}
</style>

<font style="color:#FFFFFF;"></font>

#### JS埋め込み

<script type="text/javascript">
</script>

-->


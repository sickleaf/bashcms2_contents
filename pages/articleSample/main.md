---
Keywords: bashcms2
Copyright: (C) 2021 sickleaf
articleTitle: 記事のサンプル
---

基本的にはgithub markdownの表示例を示します。

#### ヘッダ部分


```
# h1
## h2
### h3
#### h4
##### h5
###### h6
```

# h1
記事タイトルの指定。yaml部分のarticleTitleで代替して反映するため、markdown本文中にh1部分は不要。

## h2

### h3
**h3まではヘッダ部に目次のリンクが貼られる。**

#### h4
比較的見やすい区切り文字。

##### h5
最も小さい装飾。

###### h6
装飾無し、**太字**のみ。



### 本文装飾（太字、区切り）

太字は`**（太字にする箇所）**`、区切りは`---`のみの行を入力する。


### 引用表示

>で指定。

```

```

### インライン表示、コードブロック

行中のインライン表示をする場合は` `` `（バッククォート1つ） 、複数行ある場合は` ``` `（開始／終了時にバッククォート3つ）で囲む。

スタイルは`jumbotron.css`に定義しており、**`max-height: 200px`を指定して表示エリアを固定**している。

```
/* wrap code block */
pre {
        max-height: 200px;
        white-space: pre-wrap;
        white-space: moz-pre-wrap;
        white-space: -pre-wrap;
        white-space: -o-pre-wrap;
        word-wrap: break-word;
}

/* quote,code block */
code,pre > .hljs{
        background-color: #f9f6e1 !important;
        border-radius: 20px
}
```



### 記事ヘッダ部分（yamlヘッダ）
markdown冒頭に挿入。`---`を忘れないこと。

```
---
Keywords: 
Copyright: 
articleTitle: 
createdTime: 
modifiedTime : 
---
（記入例）
---
Keywords: keyword, keyword2
Copyright: (C) 2021 sickleaf
articleTitle: title_for_h1
createdTime: 2021-01-01 12:34:56
modifiedTime : 2021-01-01 12:34:56
---
```

`Keywords:`が無い場合、h1の表示とタグ・時刻表示部分はカットされる。（主にpageで使用する想定。template.htmlで`$Keywords`の指定有無を判定している）

```
$if(Keywords)$
	<p style="padding:5px"><strong>$created_time$</strong> (mod: $modified_time$)   [count：$views$] <br />
	tag: <span id="keywords">$Keywords$</span>
	</p>
$endif$
```


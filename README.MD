ToolGood.Words
==================
一款高性能非法词(敏感词)检测组件，附带繁体简体互换，支持全角半角互换，获取拼音首字母，获取拼音字母，拼音模糊搜索等功能。

`C#`语言，使用`StringSearchEx2.Replace`过滤，在48k敏感词库上的过滤速度超过3亿字符每秒。（cpu i7 8750h） 


csharp 文件夹说明：
```
ToolGood.PinYin.Build:          生成词的拼音
ToolGood.PinYin.WordsBuild:     生成多音词的拼音
ToolGood.Transformation.Build： 生成简体繁体转换文档，更新时文档放在同一目录下，词库参考 https://github.com/BYVoid/OpenCC
ToolGood.Words.Contrast:        字符串搜索对比
ToolGood.Words.Test:            单元测试
ToolGood.Words:                 本项目源代码
ToolGood.Words.PinYinSearch:    拼音搜索项目源代码
```
java版本已移植了`StringSearch`、`StringSearchEx`、`WordsSearch`、`WordsSearchEx`、`IllegalWordsSearch`、`StringTypoSearch`、`NumberTypoSearch`及部分`WordsHelper`

golang版本已移植了`StringSearch`、`StringSearchEx`、`WordsSearch`、`WordsSearchEx`、`IllegalWordsSearch`




## 繁体简体互换、全角半角互换、数字转成中文大写、拼音操作

``` csharp
    // 转成简体
    WordsHelper.ToSimplifiedChinese("壹佰贰拾叁億肆仟伍佰陆拾柒萬捌仟玖佰零壹元壹角贰分");
    WordsHelper.ToSimplifiedChinese("壹佰贰拾叁億肆仟伍佰陆拾柒萬捌仟玖佰零壹元壹角贰分",1);// 港澳繁体 转 简体
    WordsHelper.ToSimplifiedChinese("壹佰贰拾叁億肆仟伍佰陆拾柒萬捌仟玖佰零壹元壹角贰分",2);// 台湾正体 转 简体
    // 转成繁体
    WordsHelper.ToTraditionalChinese("壹佰贰拾叁亿肆仟伍佰陆拾柒万捌仟玖佰零壹元壹角贰分");
    WordsHelper.ToTraditionalChinese("壹佰贰拾叁亿肆仟伍佰陆拾柒万捌仟玖佰零壹元壹角贰分",1);// 简体 转 港澳繁体
    WordsHelper.ToTraditionalChinese("壹佰贰拾叁亿肆仟伍佰陆拾柒万捌仟玖佰零壹元壹角贰分",2);// 简体 转 台湾正体
    // 转成全角
    WordsHelper.ToSBC("abcABC123");
    // 转成半角
    WordsHelper.ToDBC("ａｂｃＡＢＣ１２３");
    // 数字转成中文大写
    WordsHelper.ToChineseRMB(12345678901.12);
    // 中文转成数字
    WordsHelper.ToNumber("壹佰贰拾叁亿肆仟伍佰陆拾柒万捌仟玖佰零壹元壹角贰分");
    // 获取全拼
    WordsHelper.GetPinYin("我爱中国");//WoAiZhongGuo
    // 获取首字母
    WordsHelper.GetFirstPinYin("我爱中国");//WAZG
    // 获取全部拼音
    WordsHelper.GetAllPinYin('传');//Chuan,Zhuan
``` 
`WordsHelper.ClearTranslate()` 清理缓存
 
## 非法词(敏感词)检测（字符串搜索）
非法词(敏感词)检测类：`StringSearch`、`StringSearchEx`、`StringSearchEx2`、`WordsSearch`、`WordsSearchEx`、`WordsSearchEx2`、`IllegalWordsSearch`;

* `StringSearch`、`StringSearchEx`、`StringSearchEx2`: 搜索`FindFirst`方法返回结果为`string`类型。
* `WordsSearch`、`WordsSearchEx`、`WordsSearchEx2`: 搜索`FindFirst`方法返回结果为`WordsSearchResult`类型,
`WordsSearchResult`不仅仅有关键字，还有关键字的开始位置、结束位置，关键字序号等。
* `IllegalWordsSearch`: 过滤非法词（敏感词）专用类，可设置跳字长度，默认**繁体转简体，全角转半角，忽略大小写，跳词，重复词，黑名单**，
搜索`FindFirst`方法返回为`IllegalWordsSearchResult`,有关键字，对应原文，开始、位置，黑名单类型。
* `IllegalWordsSearch`、`StringSearchEx`、`StringSearchEx2`、`WordsSearchEx`、`WordsSearchEx2` 使用`Save`、`Load`方法，可以加快初始化。
* 共同方法有：`SetKeywords`、`ContainsAny`、`FindFirst`、`FindAll`、`Replace`
* `IllegalWordsSearch`独有方法：`SetSkipWords`（设置跳词）、`SetBlacklist`（设置黑名单）。
* `IllegalWordsSearch`字段`UseIgnoreCase`：设置是忽略否大小写,必须在`SetKeywords`方法之前，注：使用`Load`方法则该字段无效。
* `StringTypoSearch` 错字搜索,英文数字搜索,不包含中文转数字,主要检测网址，推广账号。
* `NumberTypoSearch` 数字错字搜索，先使用静态方法ReplaceNumberSymbol对❿这些符号进行转化 ，然后使用本类
* `StringSearchEx2`、`WordsSearchEx2`为指针版优化版，实测时发现性能浮动比较大。

``` csharp
    string s = "中国|国人|zg人";
    string test = "我是中国人";

    StringSearch iwords = new StringSearch();
    iwords.SetKeywords(s.Split('|'));
    
    var b = iwords.ContainsAny(test);
    Assert.AreEqual(true, b);

    var f = iwords.FindFirst(test);
    Assert.AreEqual("中国", f);

    var all = iwords.FindAll(test);
    Assert.AreEqual("中国", all[0]);
    Assert.AreEqual("国人", all[1]);
    Assert.AreEqual(2, all.Count);

    var str = iwords.Replace(test, '*');
    Assert.AreEqual("我是***", str);
``` 

##### 性能对比
执行10万次性能对比，结果如下：

![10W次性能对比](10W次性能对比(20191014).png)

注:C#自带正则很慢，`StringSearchEx2.ContainsAny`是`Regex.IsMatch`效率的8.8万倍多，跟关键字数量有关。


`Regex.Matches`的运行方式跟`IQueryable`的类似，只返回`MatchCollection`,还没有计算。

`TrieFilter`,`FastFilter`来源: http://www.cnblogs.com/yeerh/archive/2011/10/20/2219035.html

 在 Find All测试中,

`FastFilter`只能检测出7个:
    [0]: "主席"
    [1]: "赵洪祝"
    [2]: "中国"
    [3]: "铁道部"
    [4]: "党"
    [5]: "胡锦涛"
    [6]: "倒台"

`StringSearch`检测出14个:
    [0]: "党"
    [1]: "党委"
    [2]: "西藏"
    [3]: "党"
    [4]: "党委"
    [5]: "主席"
    [6]: "赵洪祝"
    [7]: "中国"
    [8]: "铁道部"
    [9]: "党"
    [10]: "胡锦涛"
    [11]: "锦涛"
    [12]: "倒台"
    [13]: "黑社会"

`IllegalWordsSearch`检出15个：
    [0]: {63|党}
    [1]: {63|党委}
    [2]: {81|西藏}
    [3]: {83|党}
    [4]: {83|党委}
    [5]: {143|主席}
    [6]: {185|赵洪祝}
    [7]: {204|中国}
    [8]: {221|铁道部}
    [9]: {235|党}
    [10]: {244|胡锦涛}
    [11]: {245|锦涛}
    [12]: {323|倒台}
    [13]: {324|台}
    [14]: {364|黑社会}

`IllegalWordsSearch`比`StringSearch`多一个`台`，原因：关键字繁体转简体

插曲：在细查`Regex.Matches`神奇3ms，我发现`Regex.Matches`有一个小问题，

`Regex.Matches`只能检测出11个:
    [0]: "党"
    [1]: "西藏"
    [2]: "党"
    [3]: "主席"
    [4]: "赵洪祝"
    [5]: "中国"
    [6]: "铁道部"
    [7]: "党"
    [8]: "胡锦涛"
    [9]: "倒台"
    [10]: "黑社会"



#### 猜想
   非法词(敏感词)检测方法可用于电脑病毒检测及基因检测。
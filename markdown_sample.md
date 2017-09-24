###Uboot Makefile   1


>这是什么情况
>>好吧 我懂了

> ## 这是一个标题。
>
> 1.   这是第一行列表项。
> 2.   这是第二行列表项。
>
> 给出一些例子代码：
>
>     return shell_exec("echo $input | $markdown_script");

# 这是 H1

## 这是 H2

###### 这是 H6
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
  consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
  Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.

> Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
  Suspendisse id sem consectetuer libero luctus adipiscing.
    *   Red
  *   Green
  *   Blue

-   Red
-   Green
-   Blue
1.  Bird
2.  McHale
3.  Parish

*   A list item with a blockquote:

    > This is a blockquote
    > inside a list item.

    这是一个普通段落：

        这是一个代码区块。


* * *

***

*****

- - -

---------------------------------------


This is [an example](http://example.com/ "Title") inline link.

[This link](http://example.net/) has no title attribute.

I get 10 times more traffic from [Google] [1] than from
[Yahoo] [2] or [MSN] [3].
[1]: http://google.com/        "Google"
[2]: http://search.yahoo.com/  "Yahoo Search"
[3]: http://search.msn.com/    "MSN Search"

I get 10 times more traffic from [Google][] than from
[Yahoo][] or [MSN][].
[google]: http://google.com/        "Google"
[yahoo]:  http://search.yahoo.com/  "Yahoo Search"
[msn]:    http://search.msn.com/    "MSN Search"

*single asterisks*

_single underscores_

**double asterisks**

__double underscores__

Use the `printf()` function.

``There is a literal backtick (`) here.``

A single backtick in a code span: `` ` ``

A backtick-delimited string in a code span: `` `foo` ``

First Header  | Second Header
------------- | -------------
Content Cell  | Content Cell
Content Cell  | Content Cell



| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |


```ruby
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```

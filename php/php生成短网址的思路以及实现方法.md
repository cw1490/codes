# php生成短网址的思路以及实现方法

短网址流行的已经有一段时间了，以前做新浪微博应用的时候就有接触，但没有搞清楚，最近再次接触到这个东东，仔细研究了下，发现短网址其实也挺容易的。下面就将本次对于生成短网址的思路以及使用php生成短网址的实现方法做一下记录。
生成短网址的思路：如果把短网址还原了，你知道是个什么样子的吗？比如：
> http://www.daimajiayuan.com/sitejs-17300-1.html

对于以上这个链接，除了直接打开之外，还有一种方法打开它，如下：
> http://www. daimajiayuan.com/link.php?url=http://www.daimajiayuan.com/sitejs-17300-1.html

好了，短网址还原了实际就是这个样子的了，可能你看到新浪微博应用里面的短网址都是这个样子：
> http://t.cn/zHEYrvV

其实他还原了说不定就是这个样子：
> http://t.cn/link.php?url=http://www.daimajiayuan.com/sitejs-17300-1.html

好了，这里就说到第二步了，如何将
> http://t.cn/link.php?url=http://www.daimajiayuan.com/sitejs-17300-1.html

缩成
> http://t.cn/zHEYrvV

这个地方需要用到url重写，按照本例则可以这么重写：
```apacheconf
RewriteEngine On 
RewriteBase / 
RewriteRule ^/(.*)$ link.php?url=$1[L] 
```
这里就实现了将 ==http://t.cn/link.php?url=zHEYrvV== 转换为了 ==http://t.cn/zHEYrvV== ，缩短了不少，那么如何通过 zHEYrvV 去查找到 ==http://www.daimajiayuan.com/sitejs-17300-1.html== 这个网址并跳到这个网址上去呢？这里就用到了一个类似加密的算法了，通过算法将所有的长网址缩短成一个对应的5-6位的并且唯一字符串，并将这个对应关系存入到数据库中去。结合本例就是根据传入的参数 zHEYrvV 到数据库中去找对应的网址，找到了就 header 跳转过去。
ok，至于生成短网址的思路就是这个样子的了。
下面分享一下通过php生成短网址的那个过程（这里将长网址生成短至5-6位字符长度并且还需要是唯一的）：
```php
<?php 
function code62($x){ 
    $show=''; 
    while($x>0){ 
        $s=$x % 62; 
        if ($s>35){ 
            $s=chr($s+61); 
        }elseif($s>9&&$s<=35){ 
            $s=chr($s+55); 
        } 
        $show.=$s; 
        $x=floor($x/62); 
    } 
    return $show; 
} 
function shorturl($url){ 
    $url=crc32($url); 
    $result=sprintf("%u",$url); 
    return code62($result); 
} 
```
比如
```php
echo shorturl('http://www.daimajiayuan.com/'); 
```
将生成的一个唯一对应码为 n2Q8e ，OK，至于如何去做 url重写和数据库存储这里就不多写了，自己根据自己的情况来吧。

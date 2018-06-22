前几天在做一个需求，需要用到Smarty模板渲染服务端数据，其中一项工作是把“白银版”字段替换成“白银”，在Smarty的手册里搜到了replace的变量修饰符，拿过来做解决办法，开发环境完全ok。但是把代码推到测试机之后就出现问题了，“白银版”依旧是“白银版”，更奇怪的是，“黄金版”被替换成了“黄金”。好诡异。。。

<!--more-->
遇到bug就fix it，首先又重新看了一下Smarty官方手册对replace的介绍，
> 对变量进行简单的搜索和替换。 等同于PHP函数的 str_replace()。

先解决问题，将代码修改成了php的str_replace()方法，开发环境ok，测试环境ok。嗯，那看来官方手册的说法也不是很准确，为了搞清楚究竟是哪里出了问题，还是看一下Smarty的源码吧。

```php
function smarty_modifier_replace($string, $search, $replace)
{
    if (function_exists('mb_split')) {
        	require_once(SMARTY_PLUGINS_DIR . 'shared.mb_str_replace.php');
		    	return smarty_mb_str_replace($search, $replace, $string);
			    }
			        return str_replace($search, $replace, $string);
				} 
				```
				当php中没有安装扩展“mb_split”方法的时候，会执行str_replace()方法，这是没有问题的；但如果有这个扩展的话，就会执行smarty_mb_str_replace()方法。那么继续来看一下这个方法是怎么实现的。

				```php
				$parts = mb_split(preg_quote($search), $subject); 
				$subject = implode($replace, $parts);
				```
				这里先用被替换的字符来分割原字符串，返回一个array；implode()有些类似于js里面的join()方法，返回由替换的字符串连接数组元素的新字符串。

				php手册里的[mb_split注意事项](http://php.net/manual/zh/function.mb-split.php#108189)启发了编码可能会导致问题，所以最终的罪魁祸首也找到了，测试环境应该是因为php有安装mb_split的扩展同时又没有设置mb_regex_encoding('UTF-8')。

				所以为了避免再次出现上述问题，要么就像这次修复bug时直接采用str_replace()，要么就在执行Smarty之前设置mb_regex_encoding('UTF-8')，或者采用另外一个[regex_replace](https://www.smarty.net/docs/zh_CN/language.modifier.regex.replace.tpl)修饰符，它使用的是php的preg_replace实现的，不依赖环境。

				参考链接：
				- [smarty的replace陷阱](http://blog.csdn.net/qmhball/article/details/52688731)
				-  [php mb_split](http://php.net/manual/zh/function.mb-split.php#108189)
				- [Setting PHP default encoding to utf-8?](https://stackoverflow.com/questions/9351694/setting-php-default-encoding-to-utf-8)

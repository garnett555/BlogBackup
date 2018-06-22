今天看到一个题目，判断
```Object instanceof Object```和```Number instanceof Number```的结果。
<!--more-->
刚看到题目的时候有一点懵，说实话，在看到这个题目之前，我对```instanceof```的了解仅限于引用类型判断，用来判断一个对象实例是不是某一个类型的，而究竟是如何判断的，却没有研究过。那么为了细致地弄清楚，就从头看一下```instanceof```这个运算符。

下面引用MDN当中对```instanceof```的描述

> 
```instanceof``` The instanceof operator tests whether the prototype property of a constructor appears anywhere in the prototype chain of an object. The ```instanceof``` operator tests the presence of ```constructor.prototype``` in ```object```'s prototype chain.

也就是说，```instanceof```运算符用来检测```object```（左表达式）的原型链上面是否存在```constructor```（右表达式）的```prototype```属性指针所指向的那个对象。

下面来具体看一下```instanceof```是如何实现这样的沿原型链的检测的。

ECMA-262中对```instanceof```的定义可以用代码表述如下：

```javascript
function instanceof(L, R) {// L 表示左表达式，R 表示右表达式
 var O = R.prototype;// 取 R 的显示原型
  L = L.__proto__;// 取 L 的隐式原型
   while (true) { 
      if (L === null) 
           return false; 
	      if (O === L)// 这里重点：当 O 严格等于 L 时，返回 true 
	           return true; 
		      L = L.__proto__; 
		       } 
		       }
		       ```

		       看过了这段定义之后，是不是理解起来就简单多了呢？结合已有的JS原型继承的知识，再回过来看一下刚刚的题目```Object instanceof Object```和```Number instanceof Number```

		       ```Object instanceof Object```
		       ```javascript
		       O = Object.prototype
		       L = Object.__proto__
		       // 第一次判断 O !== L
		       L = Object.__proto__.__proto__
		       // Object.__proto === Function.prototype
		       // Function.prototype.__proto__ === Object.prototype
		       // 所以第二次判断时 O === L，返回true
		       ```

		       ```Number instanceof Number```


		       ```javascript
		       O = Number.prototype
		       L = Number.__proto__
		       // 第一次判断 O !== L
		       L = Function.prototype.__proto__ = Object.prototype
		       // 第二次判断时，O !== L
		       L = Object.prototype.__proto__ = null
		       // 第三次判断时，L === null，返回false
		       ```

		       这里附上一张原型继承的关系图，便于理解
		       ![alt](http://www.iversoning.cn/static/upload/20171125/3txrwdS9M2k6leMhIgOw.jpg)


		       ----------
		       突然又想起来一个```instanceof```相关的数组检测，是之前面试的时候被问到的，当在多个frame当中，采用```arrayInstance instanceof Array```去判断一个实例究竟是不是数组类型会有问题，因为多个frame就有多个全局环境，每个全局环境有自己的Array构造函数，当判断某一个frame中的对象实例是不是其他frame中Array构造函数的实例时，结果当然是false。所以在这种情况下，需要借助toString或者Array.isArray来完成任务。

		       ----------
		       参考链接

		       [MDN_instanceof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof)

		       [JavaScript instanceof 运算符深入剖析](https://www.ibm.com/developerworks/cn/web/1306_jiangjj_jsinstanceof/index.html)

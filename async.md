最近在做云盘footer组件的vue重构，周二完成了大部分功能，今天的任务是完成剩下的在线客服的弹窗。在线客服弹窗，是采用了第三方的服务。这是一个50+k的js文件，如果用户每次访问这个页面，都要加载一遍这个js，况且并不是每个访问这个页面的用户都需要客服服务的，所以这不是一个好的实现方案。所以，就有了优化的方向。（其实这个需求和懒加载还不一样，一开始寻求解决方法的时候有点集中在webpack code splitting和vue的异步组件上了，浪费了不少时间）

<!--more-->
具体是怎么做的其实也很简单，核心思路就是在触发click事件之后，动态添加script标签到DOM当中，这样就实现了延时加载的目的。为了确保后续操作时js文件已经加载好了，所以将这段Script Dom Element代码封在一个Promise中，在resolve中再执行后续的操作。

需求做起来还是很简单的，因为本身就是简单地需求。。。不过不能局限于需求，借这个机会可以发(复)散(习)一下js的异步加载方式。

1. Script Dom Element
2. defer
1. async

第一种方法，Script Dom Element，其实就是动态地将需要加载的script标签插入到DOM当中，浏览器为了避免lockup，会异步地通过DOM加载。我在项目中所采用的方法就是这种，虽然我要实现的只是按需加载。比如项目中的这段代码：
```javascript
    var s = document.createElement('script')
        s.onload = resolve
	    s.onerror = reject
	        s.type = 'text/javascript'
		    s.async = true
		        s.src = 'https://dl.ntalker.com/js/xn6/ntkfstat.js?siteid=kf_9551'
			    document.body.appendChild(s)
			    ```
			    第二种方法：defer。说到IE，其实也不总是让人头疼的。IE4就采用的defer属性就是一个很好的特性。带有defer属性(只有在有src属性时才生效，当年为了避免标签插入混乱，对于动态插入的script标签添加defer属性也无法生效)的script标签，会并行下载，并在所有DOM元素解析完成之后，DOMContentLoaded事件触发之前按序执行。

			    第三种方法：async，HTML5带来的新特性。这一属性与defer的不同之处在于，带有async的script标签，在其下载之后即会执行，执行顺序可能与标签声明的顺序不同。所以对于有依赖关系的js来说，async需要慎重使用。

			    ![alt](http://iversoning.cn/static/upload/20170803/_wHhm0HSk2WlpBPA73mcYfu3.jpg)

			    当然现在越来越多的模块库能够帮助我们解决异步加载的问题，比如RequireJs、CommonJs以及ECMAScript新特性module，不过这个就再好好整理一下，下一篇文章再记录一下~

			    参考链接：[https://www.html5rocks.com/en/tutorials/speed/script-loading/](https://www.html5rocks.com/en/tutorials/speed/script-loading/)

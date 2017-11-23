#ES6 中的Promise对象


## 写在前面
首先看一个demo

	$.ajax({
	    url: 'XXX',
	    success: function (data) {
			//第二个请求依赖第一个请求
	        $.ajax({
	            // 要在第一个请求成功后才可以执行下一步
	            url: 'xxx',
	            success: function (data) {
	                 // ......
	            }
	        });
	    }
	});

这种写法的函数回调，理解起来不难，但在实际开发中有一些缺点

+ 在多个回调的时候，会导致回调函数的嵌套，这就是常说的回调深渊。
+ 如果几个异步操作之间不分先后顺序时，同样需要等待上一个完成后在执行下一个。

为了解决这个问题，在ES6中`Promise`对象应运而生。
## 1.什么是Promise

`promise`是异步编程的一种解决方案，比传统的解决方法更合理和更强大。简单来说他就是一个容器，保存着未来才会结束的事件（通常是异步操作）的结果。有了`promise`对象，可以将异步操作以同步操作的流程表达出来，避免层层嵌套的回调。

当然`promise`也有一些缺点，首先他无法取消，一单新建就会立即执行，无法取消。其次，如果不设置回调函数，promise内部会抛出错误，不会反映到外部。最后，处于`pending`状态时，无法得知目前进展到哪一个阶段。

## 2.Promise的基本用法
`Promise`对象代表一个未完成，但预计将来会完成的操作。它有三种状态：

+ `pending` ：初始值，不是`fulfilled`，也不是`regected`；
+ `fulfilled` ：代表操作成功
+ `regected` ：代表操作失败

`Promise`有两种状态的改变，可从`pending`转变成另外两种状态。一旦状态改变，就不会发生变化。状态变化后`promise.then`绑定的函数就会被调用。 

	//构建Promise
	var promise = new Promise(function (resolve, reject) {
    	if (/* 异步操作成功 */) {
    	    resolve(data);
   		} else {
        	/* 异步操作失败 */
        	reject(error);
    	}
	});

我们使用new来构建一个Promise。Promise接受一个函数作为参数，该函数有两个参数分别为resolve和reject。这两个函数就是回调函数。

+ resolve函数的作用：在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；
+ reject 函数的作用：在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去；


Promise实例生成以后，可以用then方法指定resolved和reject状态的回调函数。
	
	promise.then(onFulfilled, onRejected);

	promise.then(function(data) {
  		// do something when success
	}, function(error) {
 		 // do something when failure
	});

then方法会返回一个Promise。他有两个参数，分别为Promise从pending变为fulfilled和reject时的回调函数。这两个函数都接受Promise对象传出的值作为参数。简单来说，then就是定义resolve和reject函数的。

Promise新建后悔立即执行。而then方法中指定的回调函数，将在当前脚本所有同步任务执行完才会执行。
	
	var promise = new Promise(function(resolve, reject) {
		console.log('before resolved');
	  	resolve();
	  	console.log('after resolved');
	});
	
	promise.then(function() {
	 	console.log('resolved');
	});
	
	console.log('outer');
	
	-------output-------
	before resolved
	after resolved
	outer
	resolved

由于resolve指定的是异步操作成功后的回调函数，它需要等所有同步代码执行后才会执行。

## 3.Promise的API

### .then()

	语法：Promise.prototype.then(onFulfilled, onRejected)

对promise添加onFulfilled和onRejected回调，并返回的是一个新的Promise实例（不是原来那个Promise实例），且返回值将作为参数传入这个心Promise的resolve函数。


### .catch()

	语法：Promise.prototype.catch(onRejected)

该方法是.then（undefined, onRejected)的别名，用于指定发生错误时的回调函数。

	promise.then(function(data) {
	    console.log('success');
	}).catch(function(error) {
	    console.log('error', error);
	});
	
	/*******等同于*******/
	promise.then(function(data) {
	    console.log('success');
	}).then(undefined, function(error) {
	    console.log('error', error);
	});

	//demo

	var promise = new Promise(function (resolve, reject) {
	    throw new Error('test');
	});
	/*******等同于*******/
	var promise = new Promise(function (resolve, reject) {
	    reject(new Error('test'));
	});
	
	//用catch捕获
	promise.catch(function (error) {
	    console.log(error);
	});
	-------output-------
	Error: test


regect方法的作用，等同于抛错。

promise对象的错误，会一直向后传递，直到被捕获。即错误总会被下一个catch所捕获。then方法指定的回调函数，若抛出错误，也会被下一个catch捕获。catch中也能抛错，则需要后面的catch来捕获。

如果promise一旦fulfilled了，再抛错，也不会变为rejected,就不会被catch了。

### .all()
	
	语法：Promise.all(iterable)

该方法用于将多个Promise实例，包装成一个新的Promise实例。

	var p = Promise.all([p1,p2,p3]);

Promise.all方法接受一个数组（或具有Iterator接口）作参数，数组中的对象（p1、p2、p3）均为promise实例（如果不是一个promise，该项会被用Promise.resolve转换为一个promise)。它的状态由这三个promise实例决定

+ 当p1, p2, p3状态都变为fulfilled，p的状态才会变为fulfilled，并将三个promise返回的结果，按参数的顺序（而不是 resolved的顺序）存入数组，传给p的回调函数.demo1
+ 当p1, p2, p3其中之一状态变为rejected，p的状态也会变为rejected，并把第一个被reject的promise的返回值，传给p的回调函数. demo2


		/* demo1 */
		var p1 = new Promise(function (resolve, reject) {
		    setTimeout(resolve, 3000, "first");
		});
		var p2 = new Promise(function (resolve, reject) {
		    resolve('second');
		});
		var p3 = new Promise((resolve, reject) => {
		  setTimeout(resolve, 1000, "third");
		}); 
		
		Promise.all([p1, p2, p3]).then(function(values) { 
		  console.log(values); 
		});
		
		-------output-------
		//约 3s 后
		["first", "second", "third"]


		/* demo2 */
		var p1 = new Promise((resolve, reject) => { 
		  setTimeout(resolve, 1000, "one"); 
		}); 
		var p2 = new Promise((resolve, reject) => { 
		  setTimeout(reject, 2000, "two"); 
		});
		var p3 = new Promise((resolve, reject) => {
		  reject("three");
		});
		
		Promise.all([p1, p2, p3]).then(function (value) {
		    console.log('resolve', value);
		}, function (error) {
		    console.log('reject', error);    // => reject three
		});
		
		-------output-------
		reject three

### .race()

	语法： Promise.race(iterable)

该方法同样是将多个Prmise实例，包装成一个新的Promise实例。
	
	var p = Promise.race([p1, p2, p3]);


### .resolve()
语法：

	Promise.resolve(value);
	Promise.resolve(promise);	
	Promise.resolve(thenable);

它可以看做new Promise()的快捷方式；

	Promise.resolve('Success');
	
	/*******等同于*******/
	new Promise(function (resolve) {
	    resolve('Success');
	});

这段代码会让这个Promise对象立即进入`resolved`状态，并将结果`success`传递给`then`指定的`onFulfilled`回调函数。用于`Promise.resolve()`也是返回Promise对象，因此可以用`.then()`处理其返回值。

	Promise.resolve('success').then(function (value) {
	    console.log(value);
	});
	-------output-------
	Success

`Promise.resolve()`的另一个作用就是将`thenable`对象（即带有`then`方法的对象）转换为promise对象。

	var p1 = Promise.resolve({ 
	    then: function (resolve, reject) { 
	        resolve("this is an thenable object!");
	    }
	});
	console.log(p1 instanceof Promise);     // => true
	
	p1.then(function(value) {
	    console.log(value);     // => this is an thenable object!
	  }, function(e) {
	    //not called
	});

### .reject()

	语法：Promise.reject(reason)

这个方法和上述的`Promise.resolve()`类似，它也是`new Promise（）`的快捷方式。

	Promise.reject(new Error('error'));
	
	/*******等同于*******/
	new Promise(function (resolve, reject) {
	    reject(new Error('error'));
	});

这段代码会让这个Promise对象立即进入`rejected`状态，并将错误对象传递给`then`指定的`onRejected`回调函数。

## 4.Promise常见问题

#### 创建promise的流程：

+ 使用`new Promise(fn)`或者它的快捷方式`Promise.resolve()`、`Promise.reject()`,返回一个promise对象 
+ 在`fn`中指定异步的处理
	
	处理结果正常，调用resolve

    处理结果错误，调用reject

#### reject和catch的区别

+ promise.then(onFulfilled,onRejected) 

        在onFulfilled中发生异常的话，在onRejected中是捕获不到这个异常的。

+ promise.then(onFulfilled).catch(onRejected)

        .then中产生的异常能在.catch中捕获

#### 实例
如果在then中抛错，而没有对错误进行处理（即catch），那么会一直保持reject状态，直到catch了错误

	function taskA() {
	    console.log(x);
	    console.log("Task A");
	}
	function taskB() {
	    console.log("Task B");
	}
	function onRejected(error) {
	    console.log("Catch Error: A or B", error);
	}
	function finalTask() {
	    console.log("Final Task");
	}
	var promise = Promise.resolve();
	promise
	    .then(taskA)
	    .then(taskB)
	    .catch(onRejected)
	    .then(finalTask);
	    
	-------output-------
	Catch Error: A or B,ReferenceError: x is not defined
	Final Task

A抛错时，会按照 taskA → onRejected → finalTask这个流程来处理。A抛错后，若没有对它进行处理，如例3.7，状态就会维持rejected，taskB不会执行，直到catch了错误。

	function taskA() {
	    console.log(x);
	    console.log("Task A");
	}
	function taskB() {
	    console.log("Task B");
	}
	function onRejectedA(error) {
	    console.log("Catch Error: A", error);
	}
	function onRejectedB(error) {
	    console.log("Catch Error: B", error);
	}
	function finalTask() {
	    console.log("Final Task");
	}
	var promise = Promise.resolve();
	promise
	    .then(taskA)
	    .catch(onRejectedA)
	    .then(taskB)
	    .catch(onRejectedB)
	    .then(finalTask);
	    
	-------output-------
	Catch Error: A ReferenceError: x is not defined
	Task B
	Final Task

在taskA后多了对A的处理，因此，A抛错时，会按照A会按照 taskA → onRejectedA → taskB → finalTask这个流程来处理，此时taskB是正常执行的。

	//方法1：对同一个promise对象同时调用 then 方法
	var p1 = new Promise(function (resolve) {
	    resolve(100);
	});
	p1.then(function (value) {
	    return value * 2;
	});
	p1.then(function (value) {
	    return value * 2;
	});
	p1.then(function (value) {
	    console.log("finally: " + value);
	});
	-------output-------
	finally: 100
	
	//方法2：对 then 进行 promise chain 方式进行调用
	var p2 = new Promise(function (resolve) {
	    resolve(100);
	});
	p2.then(function (value) {
	    return value * 2;
	}).then(function (value) {
	    return value * 2;
	}).then(function (value) {
	    console.log("finally: " + value);
	});
	-------output-------
	finally: 400

第一种方法中，then的调用几乎是同时开始执行的，且传给每个then的value都是100，这种方法应当避免。方法二才是正确的链式调用。

	function task1(data) {
	    var promise = Promise.resolve(data);
	    promise.then(function(value) {
	        //do something
	        return value + 1;
	    });
	    return promise;
	}
	task1(10).then(function(value) {
	   console.log(value);          //想要得到11，实际输出10
	});
	-------output-------
	10

	/* 正确的写法 */

	function task2(data) {
	    var promise = Promise.resolve(data);
	    return promise.then(function(value) {
	        //do something
	        return value + 1;
	    });
	    return promise;
	}
	task2(10).then(function(value) {
	   console.log(value);
	});
	-------output-------
	11
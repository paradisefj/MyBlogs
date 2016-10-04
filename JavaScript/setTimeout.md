JavaScript是**单线程**语言，但是它允许通过设置超时值和间歇时间值使代码在特定的时间内运行。

### setTimeout

超时调用需要使用 window 对象的 setTimeout()方法,它接受两个参数:要执行的代码和以毫秒 表示的时间(即在执行代码前需要等待多少毫秒)。其中,第一个参数可以是一个包含 JavaScript 代码的 字符串(就和在 eval()函数中使用的字符串一样),也可以是一个函数。例如,下面对 `setTimeout()` 的两次调用都会在一秒钟后显示一个警告框。 3
```javascript
//不建议传递字符串!
setTimeout("alert('Hello world!') ", 1000);
//推荐的调用方式 setTimeout(function() {
alert("Hello world!");
}, 1000);
```
虽然这两种调用方式都没有问题,但由于传递字符串可能导致性能损失,因此不建议以字符串作为 第一个参数。

**第二个参数是一个表示等待多长时间的毫秒数,但经过该时间后指定的代码不一定会执行。**JavaScript 是一个单线程序的解释器,因此一定时间内只能执行一段代码。为了控制要执行的代码,就 有一个 JavaScript 任务队列。这些任务会按照将它们添加到队列的顺序执行。setTimeout()的第二个 参数告诉 JavaScript 再过多长时间把当前任务添加到队列中。如果队列是空的,那么添加的代码会立即 执行;如果队列不是空的,那么它就要等前面的代码执行完了以后再执行。
调用 setTimeout()之后,该方法会返回一个数值 ID,表示超时调用。这个超时调用 ID 是计划执 行代码的唯一标识符,可以通过它来取消超时调用。要取消尚未执行的超时调用计划,可以调用 clearTimeout()方法并将相应的超时调用 ID 作为参数传递给它,如下所示。
```javascript
//设置超时调用
var timeoutId = setTimeout(function() {
alert("Hello world!");
}, 1000);
//注意:把它取消 
clearTimeout(timeoutId);
```
只要是在指定的时间尚未过去之前调用 clearTimeout(),就可以完全取消超时调用。前面的代码 在设置超时调用之后马上又调用了 clearTimeout(),结果就跟什么也没有发生一样。 
> 超时调用的代码都是在**全局作用域**中执行的,因此函数中 this 的值在非严格模 式下指向 window 对象,在严格模式下是 undefined。

#### 注意
函数或者代码片段会在当前调用` setTimeout()`的线程执行完后才会被执行：
例如：
```javascript
function foo(){
console.log('foo has been called');
}
setTimeout(foo, 0);
console.log('After setTimeout');
```
结果
```javascript
After setTimeout
foo has been called
```
尽管`setTimeout`在0延时后被调用，代码只是被放在了任务队列中，等待被执行，不是立即执行。在队列中的代码执行之前，当前的代码必须执行完成，所以，结果可能和期待不相符。

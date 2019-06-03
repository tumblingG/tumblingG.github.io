---
title: rxjs的debounceTime无效问题
categories: 前端
tags: rxjs 
---
事情起因：angular中用rxjs的`debounceTime`来做一个请求防抖动功能，
使其在用户输入的时候不会频繁发送请求，后来发现这样做是不行的。
<!--more-->
# 1、debounceTime功能描述
这里官网的一段描述，详情参考：[debounceTime](https://rxjs.dev/api/operators/debounceTime)
>Emits a value from the source Observable only after a particular time span has 
>passed without another source emission.

意思是：在一个特殊的时间间隔内没有接收到源的另一个新的提交值则提交当前`Obserable`发送的值。

这里有一点需要特别注意：这里的多次提交值的源是指`同一个源`，问题就出在这里。

下面给出一个官方的例子:
```javascript
import { fromEvent } from 'rxjs';
import { debounceTime } from 'rxjs/operators';

const clicks = fromEvent(document, 'click');
const result = clicks.pipe(debounceTime(1000));
result.subscribe(x => console.log(x));
```
注意这里的源只有一个`clicks`,无论在文档中点击多少次，值都是由这个observable发出的，所以能够成功。

# angular http请求的情况
`httpClient`的请求方法会返回一个`Obserable`，如下所示：
```javascript
const getConfig = () => {
    this.http.get('/config').subscribe(res => {
        // do something here
    });
}
```
那么问题来了，如果我用`debounceTime`来防抖动，代码如下所示：
```javascript
import { fromEvent } from 'rxjs';
import { debounceTime } from 'rxjs/operators';

const getConfig = () => {
    this.http.get('/config').pipe(debounceTime(1000))
    .subscribe(res => {
        // do something here
    });
}
```
好像这样子是没什么问题，其实不然，这里的`http.get()`每次返回不同的`Obserable`，
导致`debounceTime`好像没有设置一样。
# 解决方法
`debounceTime`是不能用了，这里提供来一个简单的方法供参考：
```javascript
let timer: any;
const getConfig = () => {
    clearTimeout(timer);
    timer = setTimeout(() => {
        clearTimeout(timer);
        this.http.get('/config').subscribe(res => {
                // do something here
            });
    }, 1000);
}
```
通过使用定时器来实现，每次调用`getConfig()`方法的时候，如果时间间隔少于1s就会取消以前的请求。








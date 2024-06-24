# 微信小程序学习记录之p115 完善loading

> [!NOTE]
>
> 学习内容 尚硅谷2024微信小程序开发教程 p115

之前我们只是简单地把loading封装到request内部，面临的其中之一的问题是**loading闪烁**，下面我们来解决这个问题

## 何为loading闪烁

当两个请求之间的时间间隔较短时，例如请求A执行完成后，请求B立刻开始执行，这时屏幕上的loading提示会在消失后立刻又出现，影响用户使用体验。

## 怎样解决loading闪烁

### 原来的代码

```js
complete: () => {
          //在每一个的请求结束后，都从queue中删除一个标识
          this.queue.pop()
    	  //当queue中没有标识后，即可隐藏loading
		  this.queue.length === 0 && wx.hideLoading()
        }
```

### 改进后的代码

```js
complete: () => {

          //在每一个的请求结束后，都从queue中删除一个标识
          this.queue.pop()
		  //在队列中添加一个新标识'judgeTime'
          this.queue.length === 0 && this.queue.push('judgeTime')
          //setTimeout，指定等待一段时间后，再执行内部的语句
    	  //这个时候，JS会去执行第二个请求
          this.timerId=setTimeout(() => {
            //如果走到这一步了，说明一秒以内没有别的请求出现
            this.queue.pop()
            //判断queue是否为空
            this.queue.length === 0 && wx.hideLoading()
            clearTimeout(this.timerId)
          }, 1);
        }
```

```js
//在request()函数开始的地方添加下列语句
this.timerId&& clearTimeout(this.timerId)
```

### 解释

在A请求调用完成时，我们不要立刻调用wx.hideLoading()，而是应该如此设置：向queue中添加一个新标识'judgeTime'，并设置一个计时器，暂停A手头上剩余的工作（即wx.hideLoading()），等一秒（时间可自行设置）后再回来执行，并在执行完毕时清除这个计时器，等待的一秒期间我们先去处理B请求；

如果B请求是紧接着开始的，那么它会注意到A请求的计时器仍存在，所以B要清除这个计时器，这就导致我们无法回去继续处理A请求的计时器内部的任务了(因为A设置的计时器被B清除了，所以A请求不会隐藏loading，而又因为queue中存在一个'judgeTime',所以A请求也就不会执行wx.showLoading，页面上的loading还是A创建的)

如果B请求是超过一秒以后才开始的，那么A在计时器的一秒结束后会继续执行，弹出'judgeTime'表示，并自行清除计时器。

这样就可以做到根据计时器的时间设置来避免laoding图标消失后又突然出现。

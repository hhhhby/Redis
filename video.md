## 打印四种启动模式启动Activity的生命周期并总结
 - 具体可见视频 standard.mp4、singleTop.mp4、singleTask.mp4

 - **standard**启动模式，如视频所示，每次跳转页面都会生成一个新的activity实例。

 - **singleTop**启动模式，如视频中所示，当第二个页面再一次跳转至第二次页面时，SecondActivity直接复用栈顶的实例（当前任务栈栈顶至栈底依次为2,1），这里会触发SecondActivity的onnewintent方法

 - **singleTask**启动模式，如视频中所示，跳转顺序为FirstActivity ->SecondActivity->ThirdActivity-> SecondActivity,当第三个页面准备跳转至第二个页面时，此时任务栈栈顶至栈底依次为3,2,1。这时任务栈中已存在SecondActivity的实例，因此，3将被移除，将SecondActivity置于栈顶。

 - 当启动一个新的Activity时，首先会调用Activity生命周期的onCreate方法，接着会进入onStart方法，表示该Activity进入启动状态，对用户可见，然后调用onResume方法，此时可以与用户互动，当触发跳转页面事件时，前一个Activity会调用onPause进入暂停状态，然后当前Activity依次调用onCreate、onStart、onResume方法，前一个Activity调用onStop方法，当返回时，前一个Activity调用onRestart()、onStart、onResume方法，然后后一个会调用onStop、onDestory方法结束生命周期，再按返回，前一个Activity也会调用相同的方法结束生命周期。

## 打印startService和bindService生命周期并总结
 - 具体可见视频 service.mp4
 - **srartService**
   - 当服务第一次创建时会调用onCreate方法，每次通过StartService方法启动服务时会调用onStartCommand，服务停止时会调用onDestroy方法

 - **bindService**
   - 服务启动时会调用onCreate方法，当Activity与Service绑定时会调用onBind方法，解绑时调用onUnbind方法，并调用onDestroy方法进行销毁。


## 分别使用动态注册和静态注册广播，并接受广播日志
 - 具体可见视频 broadcast.mp4


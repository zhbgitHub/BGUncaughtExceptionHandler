# BGUncaughtExceptionHandler
> 用途:iOS端开发异常捕获,记录异常信息,上报异常,监听代码执行过程中的异常,对外开发:挂载异常监听函数,异常处理函数,错误信号处理函数,获取已记录的异常信息函数,删除一条/多条异常信息函数.

## 原理:
* NSException (BGHandler)通过运行时交互方法raise方法,达到拦截异常效果,将拦截到的异常交给'BGUncaughtExceptionHandler'处理.
* 在多核处理器的系统 如何保证 多线程中数据不会错乱,这里通过'#include<libkern/OSAtomic.h>'中函数对数据进行加锁保护.
* 如何确定崩溃信息具体在 源码的哪一行 ? 通过'#include <execinfo.h>'中函数确定具体崩溃信息.
* 根据后台/大数据要求,处理崩溃信息,处理完毕,保存至沙盒(待达到要求上报给后台,清理缓存).
* 在处理崩溃信息时,弹框提醒用户'当前程序不稳定,请重启!',然后通过runloop hold住异常,等待用户,然后结束runloop及线程.
```objc
    CFRunLoopRef runLoop = CFRunLoopGetCurrent();
    CFArrayRef allModes = CFRunLoopCopyAllModes(runLoop);
    while (!dismissed) {
        for (NSString *mode in (__bridge NSArray *)allModes) {
            NSLog(@"RunLoop 开始运行");  
            //每次RunLoop只运行10秒，每10秒做一次检测，如果没有需要处理的后台任务了，就让此线程自己终止，不用暴力Kill  
            CFRunLoopRunInMode((CFStringRef)mode, 0.001, false);
            NSLog(@"RunLoop 停止运行");  
        }
    } 
    CFRelease(allModes);//释放modes

```

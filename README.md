# android-fingerprint
>创新实践——安卓指纹识别，利用FingerprintManager主类进行指纹识别。

#### 在安卓6.0中新增了API，FingerprintManager类，它是Google提供的帮助访问指纹硬件的API类

#### [官方文档](http://www.zhdoc.net/android/reference/android/hardware/fingerprint/FingerprintManager.html)

#### [Github项目地址](https://github.com/CasterWx/android-fingerprint)

#### 新增API权限的过程如下

       ContextCompact.checkSelfPermission  // 检查APP是否拥有某权限
       ActivityCompat.requestPermissions()  // 如果没有就去申请
       onRequestPermissionResult()  //异步执行回调结果
       ActivityCompat.shouldShowRequestPermissionRationale // 用于给用户解释权限用途

#### AndroidManifest权限声明
     <uses-permission android:name="android.permission.USE_FINGERPRINT"/>

#### FingerprintManager类
   三个主要方法
   1. authenticate(...) 启动指纹识别
   2. hasEnrolledFingerprints() 判断是否录入有指纹
   3. isHardwareDetected() 判断是否有硬件支持
   
   
#### 实现要点
1 . 判断是否硬件支持 
```
    if (!mManager.isHardwareDetected()) {
       Toast.makeText(mContext, "没有指纹识别模块", Toast.LENGTH_SHORT).show();
       return false;
     }
``` 
2 . 检查手机是否已录入指纹
```
if (!mManager.hasEnrolledFingerprints()) {
    Toast.makeText(mContext, "没有指纹录入", Toast.LENGTH_SHORT).show();
    return false;
}
```
3 . 创建指纹开启的回调方法

>这里就该引入上面所说的FingerprintManager的三个内部类了

>①FingerPrintManager.AuthenticationCallback：
在验证时传入该接口，通过该接口来返回验证指纹的结果

>②FingerPrintManager.AuthenticationResult：
当指纹验证正确时，接口里返回的参数

>③FingerPrintManager.CryptoObject：
由FingerPrintManager支持的封装加密对象的类

只要指纹识别的结果，只需要AuthenticationCallback类即可。
这一步我们就创建AuthenticationCallback类对象。
```
FingerprintManager.AuthenticationCallback mSelfCancelled = new FingerprintManager.AuthenticationCallback() {
    @Override
    public void onAuthenticationError(int errorCode, CharSequence errString) {
        //多次指纹密码验证错误后，进入此方法；并且，不可再验（短时间）
        //errorCode是失败的次数
        ToastUtils.show(mContext, "尝试次数过多，请稍后重试", 3000);
    }

    @Override
    public void onAuthenticationHelp(int helpCode, CharSequence helpString) {
        //指纹验证失败，可再验，可能手指过脏，或者移动过快等原因。
    }

    @Override
    public void onAuthenticationSucceeded(FingerprintManager.AuthenticationResult result) {
        //指纹密码验证成功
    }

    @Override
    public void onAuthenticationFailed() {
        //指纹验证失败，指纹识别失败，可再验，错误原因为：该指纹不是系统录入的指纹。
    }
};
```
4 . 开启指纹识别

只需要传参即可。
```
mManager.authenticate(null, mCancellationSignal, 0, mSelfCancelled, null);
```


## 扩展小猫粮：
### 一. authenticate参数说明
```
/**
 * 参数说明：
 * FingerprintManager.CryptoObject - 用于通过指纹验证取出AndroidKeyStore中的key的对象，用于加密
 * CancellationSignal - 用来取消指纹验证，如果想手动关闭验证，可以调用该参数的cancel方法
 * int - 没什么意义，就是传0就好了
 * FingerprintManager.AuthenticationCallback -  最重要，由于指纹信息是存在系统硬件中的，app是不可以访问指纹信息的，所以每次验证的时候，系统会通过这个callback告诉你是否验证通过、验证失败等
 * Handler - FingerPrint中的消息都通过这个Handler来传递消息，如果你传空，则默认创建一个在主线程上的Handler来传递消息，没什么用，传null好了
 */
public void authenticate(FingerprintManager.CryptoObject crypto, CancellationSignal cancel, int flags, FingerprintManager.AuthenticationCallback callback, Handler handler)
```
### [二. 什么是生命？什么是人工智能？](https://www.cnblogs.com/LexMoon/p/srgzn.html)
> 但我们今天不站队，而是从另外一个“诡异”视角，去审视一下什么是生命，什么是人工智能

### [三. 双重空间，梦境==现实？](https://www.cnblogs.com/LexMoon/p/skgn.html)
> 想象一下你获得了一种能力——你的梦境是连续的，每天睡着之后，你都会来到一个与现实世界不同、但与前一天的梦境相同的环境中。
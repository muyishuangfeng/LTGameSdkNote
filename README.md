# LTGameSdkNote
### 支付流程说明

    为了保证支付的安全，sdk采用了Android支付平台（Google Play和OneStore）验证和LT服务器验证两种方式，业务流程图如下：

![TIM图片20190118111718.png](https://upload-images.jianshu.io/upload_images/1716569-d609964d9efb2856.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### GooglePlay 接入

(1)、客户端调用支付模块代码，由SDK向LT服务器发起一个订单请求；
(2)、拿到服务器给的订单ID后，SDK发起Google Play支付；
(3)、Google Play支付返回结果后，拿到Google Play支付成功的token信息；
(4)、SDK把Google play返回的订单号token等信息发给服务端，LT服务器向Google Play服务器校验；
(5)、服务端对Google play服务器响应数据做处理和校验订单的合法性；
(6)、通过服务器验证的结果来确定支付的结果；
(7)、支付成功回调 支付成功回调的使用方法： 需要CP服务器建立一个API供乐推服务调用 回调地址所接受的参数格式
~~~
 METHOD: POST 
 HEADER:Content-Type:application/json
~~~
 + BODY:
__1、初始化参数说明__ 

|参数|类型|是否必须|说明|
|:------:|:------:|:-----:|:------|
|context| Context|是|上下文|
|publicKey|String|是|Google Play 生成的公钥|
|OnGoogleInitListener|Interface|是|初始化接口Google回调接口|

__2、支付请求参数说明__

|参数|类型|是否必须|说明|
|:-------:|:-------:|:-------:|:-----:|
|context| Context|是|上下文|
|LT-AppID|String|是|每个应用对应的appid|
|LTAppKey|String|是|每个应用对应的appKey|
|packageId|String|是|每个应用的包名|
|gid|String|是|服务器配置的id|
|custom|Map<String,Object>|是|游戏上自定义数据，可以包含充值的区服等信息|
|requestCode|String|是|支付请求码|
|goodsList|List<String>|是|内购商品集合|
|productID|String|是|内购商品唯一ID|
|OnGooglePayResultListener|Interface|是|支付接口回调接口|


__3、支付结果回调参数说明__

|参数|类型|是否必须|说明|
|:-------:|:-------:|:-------:|:----:|
|context| Context|是|上下文|
|requestCode|int|是|对应onActivityResult方法中的requestCode|
|data|Intent|是|对应onActivityResult方法中的data|
|selfRequestCode|int|是|自定义的请求码和支付请求中的requestCode保持一致|
|LT-AppID|String|是|每个应用对应的appid|
|LTAppKey|String|是|每个应用对应的appKey|
|OnGoogleResultListener|Interface|是|支付结果回调接口|

#### Google Play支付简介

+ 1、ltgamegoogle jar包；
+ 2、GooglePlayManager 封装类；
+ 3、 回调接口：
   + 1、 OnGoogleInitListener Google Play 初始化接口

         /**
          * 初始化成功
          *
          * @param success 成功回调信息
          */
         void onGoogleInitSuccess(String success);
        
         /**
          * 初始化失败回调
          *
          * @param result 失败回调信息
          */
         void onGoogleInitFailed(String result);

 + 2、OnGooglePayResultListener 
 
     支付结果回调接口

      /**
       * 支付成功 
       *
       * @param result 成功信息
       */

        void onPaySuccess(String result); 


       /**
         * 支付失败
         *
         * @param result 失败信息 
         */

        void onPayFailed(Throwable ex);
        
      /**
        * 支付完成
        *
        */
      void onPayComplete();

      /**
       * 支付错误
       *
       * @param result 错误信息
       */
      void onPayError(String result);

+ 3、OnGoogleResultListener Google Play 服务器返回信息

      /**
       * Google play 返回信息（成功）
       * 
       * @param result 成功信息
       */ 
      void onResultSuccess(String result);

       /** 
        * 上传到服务器错误 
        *
        * @param ex 错误信息 
        */

      void onResultError(Throwable ex);
      /** 
       * Google play 返回信息（失败）
       * 
       * @param failedMsg 失败内容 
       */

       void onResultFailed(String failedMsg);

#### Google Play 支付接入（Android studio）

  + 1、接入前提 
    __手机必须支持Google 服务并且已经登录__

2、在项目的build下面引用仓库地址

     maven { url 'https://jitpack.io' }
![4](https://upload-images.jianshu.io/upload_images/1716569-84f44d0667d0283a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



  + 3、在所使用的moule的 app.build中添加项目引用

         implementation 'com.github.muyishuangfeng:LTGameSdkGooglePlay:1.0.8'




 + 4、在需要调用google play支付的Activity或者Fragment中调用 __初始化__、__支付和__支付回调__的方法

        1、初始化方法
           GooglePlayManager.getInstance().initGooglePay(this, "公钥",
                new OnGoogleInitListener() {
                    @Override
                    public void onGoogleInitSuccess(String success) {
                        Log.e(TAG, success);
                    }

                    @Override
                    public void onGoogleInitFailed(String result) {
                        Log.e(TAG, result);
                    }
                });
         2、支付方法
         
                 Map<String, Object> params = new WeakHashMap<>();
                 params.put("xx", "xx");
                 params.put("xx", "xx");
                //所有内购商品集合（在Google Play Console中配置的商品）
                List<String> oldSkus = new ArrayList<>();
                oldSkus.add("xxx");
                oldSkus.add("xxx");
                oldSkus.add("xxx");
                GooglePlayManager.getInstance().getRecharge(
                        this, LTAppID,LTAppKey,packageName,gid,params,
                        oldSkus, productID, selfRequestCode, new OnGooglePayResultListener() {
                            @Override
                            public void onPaySuccess(String result) {
                                Log.e(TAG, result);
                            }
         
                            @Override
                            public void onPayFailed(Throwable ex) {
         
                            }
         
                            @Override
                            public void onPayComplete() {
         
                            }
         
                            @Override
                            public void onPayError(String result) {
         
                            }
                        }
                );
          3、支付回调方法
            
            @Override
            protected void onActivityResult(int requestCode, int resultCode, Intent data) {
            super.onActivityResult(requestCode, resultCode, data);
            GooglePlayManager.getInstance().onActivityResult(requestCode,  data, selfRequestCode,
                LTAppID, LTAppKey,
                new OnGoogleResultListener() {
                    @Override
                    public void onResultError(Throwable ex) {
                        Log.e(TAG,ex.getMessage());
                    }
         
                    @Override
                    public void onResultFailed(String failedMsg) {
                        Log.e(TAG,failedMsg);
                    }
         
                    @Override
                    public void onResultSuccess(String result) {
                        Log.e(TAG,result);
                    }
                });
        }

         4、释放资源
         
         @Override
         protected void onDestroy() {
         super.onDestroy();
         GooglePlayManager.getInstance().release();
       }

 + 5 、Google play 配置参考
 [Google Play 配置参考](https://blog.csdn.net/alex_my/article/details/82984706#2__8)

 + 6、结果码（具体以接口回调结果为准）

       1、 init Success 初始化成功
       2、 Order creation failed 订单创建失败
       3、 Purchase successful 支付成功
       4、 Purchase Failed 支付失败
       5、 ltOrderID is null 乐推订单是空
       6、 200成功
    
----
****
 __分割线__
****
----

#### OneStore 接入

##### 前提：
  手机上必须支持OneStore服务（安装OneStore客户端）

__参数说明__

 + 1、初始化参数说明
   

   |参数|类型|是否必须|说明|
|:-------:|:-------:|:-------:|:----:|
|context| Context|是|上下文|
|publicKey|String|是|OneStore平台分配的publicKey|
|LT-AppID|String|是|每个应用对应的appid|
|LTAppKey|String|是|每个应用对应的appKey|
|packageId|String|是|在OneStore配置的包名|
|gid|String|是|服务配置的商品ID|
|params|Map<String,Object>|是|游戏上自定义数据，可以包含充值的区服等信息|
|onOneStoreSupportListener|Interface|是|是否支持oneStore支付的接口回调|
|onOneStoreUploadListener|Interface|是|上传到服务器验证订单的接口回调|

+ 2、支付（购买）参数说明

|参数|类型|是否必须|说明|
|:-------:|:-------:|:-------:|:----:|
|context| Context|是|上下文|
|selfRequestCode|int|是|支付请求码|
|productName|String|否|商品名称|
|productId|String|是|商品的唯一ID（在OneStore中配置的）|
|onOneStoreSupportListener|Interface|是|是否支持oneStore的接口回调|


+3、支付（购买）结果回调参数说明

|参数|类型|是否必须|说明|
|:-------:|:-------:|:-------:|:----:|
|requestCode|int|是|onActivityResult对应的requestCode参数|
|resultCode|int|是|onActivityResult对应的resultCode参数|
|data|Intent|是|onActivityResult对应的data参数|
|selfRequestCode|int|是|请求码（和支付的请求码对应起来）|

#### OneStore支付简介

1、LTGameOneStore jar包；

2、OneStorePayManager 封装类；

3、OneStoreResult 回调状态类；

3、 回调接口：

  + 1、 onOneStoreSupportListener  查询是否支持oneStore支付的接口


        /**
         * oneStore连接失败
        *
        * @param failedMsg 失败信息
        */
      void onOneStoreClientFailed(String failedMsg);
    
      /**
       * oneStore支付失败
       *
       * @param result 失败信息
       */
      void onOneStoreFailed(OneStoreResult result);
    
      /**
       * oneStore支付出错
       *
       * @param result 错误信息
       */
      void onOneStoreError(String result);
    
      /**
       * oneStore支付成功
       *
       * @param result 成功信息
       */
      void onOneStoreSuccess(OneStoreResult result);
    
      /**
       * oneStore连接成功
       */
      void onOneStoreConnected();
    
      /**
       * oneStore未连接成功
       */
      void onOneStoreDisConnected();

 + 2、 onOneStoreUploadListener  上传到服务器验证支付的接口

        /**
     ​    * 上传验证成功
     ​    *
     ​    * @param result 成功code
     ​    */
       void onOneStoreUploadSuccess(int result);

       /**
        * 上传验证失败
           *
        * @param error 失败信息
           */

         void onOneStoreUploadFailed(Throwable error);
  + 3、OnCreateOrderFailedListener 创建订单回调
  
         /**

     ​    * 创建失败回调
     ​    *
     ​    * @param failed 失败内容
     ​    */
         void onCreateOrderFailed(String failed);
         /**
     ​    * 创建错误回调
     ​    *
     ​    * @param errorMsg 错误内容
     ​    */
         void onCreateOrderError(String errorMsg);

### OneStore  支付接入（Android studio）
1、接入前提

__手机必须支持OneStore 服务、安装OneStore客户端并且已经登录__
2、在项目的build下面引用仓库地址

     maven { url 'https://jitpack.io' }
![库地址.png](https://upload-images.jianshu.io/upload_images/1716569-fe91ec2fd8ac178c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


3、在所使用的moule的 app.build中添加项目引用

    implementation 'com.github.muyishuangfeng:LTGameSdkOneStore:1.0.4'




 4、在需要调用OneStore支付的Activity或者Fragment中调用初始化、支付、支付回调和释放资源的方法

       1、初始化方法      
        Map<String, Object> params = new WeakHashMap<>();
        params.put("xx", "xxx");
        params.put("xxx", "xxxx");
        OneStorePayManager.getInstance(this,公钥)
                .initOneStore(this, LTAppID,
                        LTAppKey, 在OneStore配置的包名,gid, params, new 
                 onOneStoreSupportListener() {
                            @Override
                            public void onOneStoreClientFailed(String failedMsg) {
                                Log.e("oneStoreINIT", failedMsg);
                            }
    
                            @Override
                            public void onOneStoreFailed(OneStoreResult result) {
                                Log.e("oneStoreINIT", result.getCode() + "==" + result.getDescription());
                            }
    
                            @Override
                            public void onOneStoreError(String result) {
                                Log.e("oneStoreINIT", result);
                            }
    
                            @Override
                            public void onOneStoreSuccess(OneStoreResult result) {
                                Log.e("oneStoreINIT", result.getCode() + "==" + result.getDescription());
                            }
    
                            @Override
                            public void onOneStoreConnected() {
                                Log.e("oneStoreINIT", "onOneStoreConnected");
                            }
    
                            @Override
                            public void onOneStoreDisConnected() {
                                Log.e("oneStoreINIT", "onOneStoreDisConnected");
                            }
                        },
                        new OnCreateOrderFailedListener() {
                            @Override
                            public void onCreateOrderFailed(int failed) {
    
                            }
    
                            @Override
                            public void onCreateOrderError(String errorMsg) {
    
                            }
                        },new onOneStoreUploadListener() {
                            @Override
                            public void onOneStoreUploadSuccess(int result) {
    
                            }
    
                            @Override
                            public void onOneStoreUploadFailed(Throwable error) {
    
                            }
                        });
    
              2、支付（购买）方法
              OneStorePayManager.getInstance(
                     OneStoreActivity.this,公钥)
                     .buyProduct(上下文,
                             自定义的请求码,
                             商品名,
                             商品ID,
                             new onOneStoreSupportListener(){
                                 @Override
                                 public void onOneStoreClientFailed(String failedMsg) {
                                     Log.e("oneStore",failedMsg);
                                 }
    
                                 @Override
                                 public void onOneStoreFailed(OneStoreResult result) {
                                     Log.e("oneStore",result.getCode()+"=="+result.getDescription());
                                 }
    
                                 @Override
                                 public void onOneStoreError(String result) {
                                     Log.e("oneStore",result);
                                 }
    
                                 @Override
                                 public void onOneStoreSuccess(OneStoreResult result) {
                                     Log.e("oneStore",result.getCode()+"=="+result.getDescription());
                                 }
    
                                 @Override
                                 public void onOneStoreConnected() {
                                     Log.e("oneStore","onOneStoreConnected");
                                 }
    
                                 @Override
                                 public void onOneStoreDisConnected() {
                                     Log.e("oneStore","onOneStoreDisConnected");
                                 }
                             });
            }
        });
       
      3、支付回调
        @Override
        protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data)；
       OneStorePayManager.getInstance(this,PUBLIC_KEY).onActivityResult(requestCode,
       resultCode,data, selfRequestCode);
       }
    4、释放资源
    
        @Override
        protected void onDestroy() {
        super.onDestroy();
        OneStorePayManager.getInstance(this, PUBLIC_KEY).release();
    }


​    
+ 5、参考配置
[OneStore 配置参考](https://dev.onestore.co.kr/devpoc/reference/view/IAP_v17_cn)
[OneStore github参考](https://github.com/ONE-store/iap_v5/tree/onestore_iap_sample)
  
#### 结果码

    RESULT_PAY_OK(0, "Payment success") 支付成功
    RESULT_USER_CANCELED(1, "User cancel") 用户取消
    RESULT_SERVICE_UNAVAILABLE(2, "service unavailable") 未连接上服务器
    RESULT_BILLING_UNAVAILABLE(3, "billing unavailable") 不支持oneStore支付
    RESULT_ITEM_UNAVAILABLE(4, "item unavailable") 商品不可用
    RESULT_DEVELOPER_ERROR(5, "developer error")开发账户异常
    RESULT_CONNECTED_SUCCESS(6, "PurchaseClient connect success") 连接成功
    RESULT_DISCONNECT_ERROR(7, "PurchaseClient Disconnect") 连接失败
    RESULT_CONNECTED_NEED_UPDATE(8, "Connected need update") oneStore客户端需要更新
    RESULT_BILLING_OK(9, "Billing success") 支持支付
    RESULT_BILLING_REMOTE_ERROR(10, "Billing Remote Connection Failed") 查询是否支持oneStore服务时提示远端服务器连接异常（未连接到服务器）
    RESULT_BILLING_SECURITY_ERROR(11, "Billing Apply State exceptions")查询是否支持oneStore服务时提示支付状态异常
    RESULT_BILLING_NEED_UPDATE(12, "Billing need update")查询是否支持oneStore服务时提示oneStore客户端需要更新
    RESULT_PURCHASES_OK(13, "Purchases success")查询购买信息成功
    RESULT_PURCHASES_REMOTE_ERROR(14, "Purchases Remote Connection Failed")查询购买信息时提示oneStore远程服务器连接异常（未连接到服务器）
    RESULT_PURCHASES_SECURITY_ERROR(15, "Purchases Apply State exceptions")查询购买信息时提示支付状态异常
    RESULT_PURCHASES_NEED_UPDATE(16, "Purchases need update")查询购买信息时提示oneStore客户端需要更新
    RESULT_CONSUME_OK(17, "Consume success")消费成功
    RESULT_CONSUME_REMOTE_ERROR(18, "Consume Remote Connection Failed")购买时提示远端服务器连接异常（未连接到oneStore）
    RESULT_CONSUME_SECURITY_ERROR(19, "Consume Apply State exceptions")购买时提示支付状态异常
    RESULT_CONSUME_NEED_UPDATE(20, "Consume need update")购买时提示oneStore客户端需要更新
    RESULT_SIGNATURE_FAILED(21, "Signature failed")签名异常
    RESULT_PURCHASES_FLOW_OK(22, "Purchases Flow success")购买成功
    RESULT_PURCHASES_FLOW_REMOTE_ERROR(23, "Purchases Flow Remote Connection 
    Failed")购买时提示远端服务器连接异常（未连接到服务器）
    RESULT_PURCHASES_FLOW_SECURITY_ERROR(24, "Purchases Flow Apply State 
     exceptions")购买时提示支付状态异常
    RESULT_PURCHASES_FLOW_NEED_UPDATE(25, "Purchases Flow need update")购买时提 
    示oneStore需要更新

### 前提

### QQ 登录接入说明

+ 1、在项目的build下面引用仓库地址

       maven { url 'https://jitpack.io' }

   ![4](https://upload-images.jianshu.io/upload_images/1716569-84f44d0667d0283a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


+ 2、在需要QQ登录的项目的配置文件（app.build）中添加引用

       implementation 'com.github.muyishuangfeng:LTGameSdkQQ:1.0.2'




+ 3、在项目的配置文件中（app.build）配置QQ的appID，如下图所示

![QQ的AppID.png](https://upload-images.jianshu.io/upload_images/1716569-7f218622a6089847.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 4、参数说明

|参数|类型|是否必须|说明|
|:------:|:------:|:-----:|:------|
|context| Context|是|上下文|
|appID|String|是|QQ平台申请的AppID|
|LT-AppID|String|是|每个应用对应的appid|
|LTAppKey|String|是|每个应用对应的appKey|
|OnLoginSuccessListener|Interface|是|登录接口|

  + 5、接口说明
  
    public interface OnLoginSuccessListener {
    /**
     * 成功回调
     *
     * @param result 成功信息
     */
    void onSuccess(BaseEntry<ResultData> result);

    /**
     * 请求失败
     *
     * @param ex 失败信息
     */
    void onFailed(Throwable ex);

    /**
     * 完成时
     */
    void onComplete();

    /**
     * 参数错误
     *
     * @param result 错误信息
     */
    void onParameterError(String result);

    /**
     * 错误回调
     *
     * @param error 错误信息
     */
    void onError(String error);

    }


  + 6、登录类引用说明
    ​     
    
        1、开始登录
        QQLoginManager.qqLogin(this, appID, LTAppID, LTAppKey,
      ​                   new OnLoginSuccessListener() {
      ​                      @Override
      ​                      public void onSuccess(BaseEntry<ResultData> result) {

                            }
        
                            @Override
                            public void onFailed(Throwable ex) {
        
                            }
        
                            @Override
                            public void onComplete() {
        
                            }
        
                            @Override
                            public void onParameterError(String result) {
        
                            }
        
                            @Override
                            public void onError(String error) {
        
                            }
                        }
                );
            }
        });

     2、登录回调
    ​    @Override
    ​    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
    ​      QQLoginManager.onActivityResultData(requestCode, resultCode, data);
    ​      super.onActivityResult(requestCode, resultCode, data);
    }
-----
****
分割线
****
-----

### 微信登录接入说明

  + 1、在项目的build下面引用仓库地址

         maven { url 'https://jitpack.io' }
![4](https://upload-images.jianshu.io/upload_images/1716569-84f44d0667d0283a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    //用以回调微信登录返回信息
      implementation 'org.greenrobot:eventbus:3.1.1'
      implementation 'com.github.muyishuangfeng:LTGameSdkWeChat:1.0.1'




+ 2、参数说明

|参数|类型|是否必须|说明|
|:------:|:------:|:-----:|:------|
|appID| String|是|微信平台申请的AppID|
|appSecret|String|是|微信平台分配的appSecret|
|LT-AppID|String|是|每个应用对应的appid|
|LTAppKey|String|是|每个应用对应的appKey|

 + 3、登录类引用

       1、在onCreate中注册EventBus
    ​     EventUtils.register(this);
    
       2、调用登录方法
       WeChatLoginManager.getInstance(this).weChatLogin(
    ​                    appID, appSecret,
    ​                    LTAppID, LTAppKey);

        3、登录结果回调
          /**
           * 获取微信回调信息
           *
           * @param event
           */
        @Subscribe
        protected void receiveEvent(Event event) {
           switch (event.getCode()) {
            case BaseResult.MSG_RESULT_ERR_AUTH_DENIED: {
        
                break;
            }
            case BaseResult.MSG_RESULT_ERR_BAN: {
                Log.e("weChat", "MSG_RESULT_ERR_BAN");
                break;
            }
            case BaseResult.MSG_RESULT_ERR_COMM: {
                Log.e("weChat", "权限不够");
                break;
            }
            case BaseResult.MSG_RESULT_ERR_PARAMETER: {
        
                break;
            }
            case BaseResult.MSG_RESULT_ERR_USER_CANCEL: {
        
                break;
            }
            case BaseResult.MSG_RESULT_SUCCESS: {
        
                break;
            }
        }
       }
    ​    4、在onDestory反注册EventBus
    ​      EventUtils.unregister(this);

+ 4、 回到结果说明
  
       MSG_RESULT_ERR_AUTH_DENIED //用户拒绝授权
       MSG_RESULT_ERR_USER_CANCEL //用户取消
       MSG_RESULT_ERR_UNSUPPORT //不支持
       MSG_RESULT_ERR_BAN //微信平台配置签名不对
       MSG_RESULT_ERR_COMM//没有微信登录权限（必须用可以进行微信登录权限的账号申请）
      MSG_RESULT_SUCCESS//登录成功

-----
****
分割线
****
----
### facebook登录接入说明
  + 1、配置

      1）、在项目的build下面引用仓库地址

         maven { url 'https://jitpack.io' }
![4](https://upload-images.jianshu.io/upload_images/1716569-84f44d0667d0283a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
       2）、在项目的app.build中引用facebook的网络包如下所示

     implementation 'com.github.muyishuangfeng:LTGameSdkFaceBook:1.0.2'

   3）、在项目的清单文件（Manifest）中配置
     
         <!-- facebook登录 -->
        <meta-data
            android:name="com.facebook.sdk.ApplicationId"
            android:value="@string/facebook_app_id" />
    
        <activity
            android:name="com.facebook.FacebookActivity"
            android:configChanges="keyboard|keyboardHidden|screenLayout|screenSize|orientation"
            android:label="@string/app_name" />
        <activity
            android:name="com.facebook.CustomTabActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
    
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
    
                <data android:scheme="@string/fb_login_protocol_scheme" />
            </intent-filter>
        </activity>

__注意:facebook_app_id为facebook平台申请的appID，fb_login_protocol_scheme为facebook平台申请的scheme__
​      4）、在app.build中配置appID和scheme


+ 2、参数说明

|参数|类型|是否必须|说明|
|:------:|:------:|:-----:|:------|
|context| Context|是|上下文|
|LT-AppID|String|是|每个应用对应的appid|
|LTAppKey|String|是|每个应用对应的appKey|
|OnLoginSuccessListener|Interface|是|登录回调接口|

+ 3、接口说明

       /**
        * 登录回调
        */
      public interface OnLoginSuccessListener {
      /**
   ​    * 成功回调
   ​    *
   ​    * @param result 成功信息
   ​    */
      void onSuccess(BaseEntry<ResultData> result);

      /**
   ​    * 请求失败
   ​    *
   ​    * @param ex 失败信息
   ​    */
      void onFailed(Throwable ex);

      /**
   ​    * 完成时
   ​    */
      void onComplete();

      /**
   ​    * 参数错误
   ​    *
   ​    * @param result 错误信息
   ​    */
      void onParameterError(String result);

      /**
   ​    * 错误回调
   ​    *
   ​    * @param error 错误信息
   ​    */
      void onError(String error);

+ 4、登录方法说明

       1、初始化
       FaceBookLoginManager.getInstance().initFaceBook(this, 
                "", "",
                new OnLoginSuccessListener() {
                    @Override
                    public void onSuccess(BaseEntry<ResultData> result) {
                        Log.e("facebook", result.getResult());
                    }
        
                    @Override
                    public void onFailed(Throwable ex) {
                        Log.e("facebook", ex.getMessage());
                    }
        
                    @Override
                    public void onComplete() {
                        Log.e("facebook", "===onComplete");
                    }
        
                    @Override
                    public void onParameterError(String result) {
                        Log.e("facebook", result);
                    }
        
                    @Override
                    public void onError(String error) {
                        Log.e("facebook", error);
                    }
                });
             2、facebook登录
             FaceBookLoginManager.getInstance().faceBookLogin(FaceBookActivity.this);
             3、facebook回调（在onActivityResult下引用）
           FaceBookLoginManager.getInstance().setOnActivityResult(requestCode, resultCode, data);

+ 5、结果码说明（具体以返回结果为准）
     ​    "OK"或者200表示成功
     ​    "NO"表示登录失败
     ​    "onCancel"表示登录取消
     ​    
-----
****
  分割线
****
----
### Google登录接入说明

  + 1、配置
      1）、在项目的build下面引用仓库地址

         maven { url 'https://jitpack.io' }
![4](https://upload-images.jianshu.io/upload_images/1716569-84f44d0667d0283a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
       2）、在项目的app.build中引用google的网络包如下所示：

        implementation 'com.github.muyishuangfeng:LTGameSdkGoogle:1.0.4'



__注意: 如果之前接入了Google Play支付不可重复配置__
+ 2、参数说明

     1）、登录参数

|参数|类型|是否必须|说明|
|:------:|:------:|:-----:|:------|
|context| Context|是|上下文|
|server_client_id| String|是|google平台申请的clientID|
|selfRequestCode|int|是|请求码（自定义）|

   2）、回调参数
|参数|类型|是否必须|说明|
|:------:|:------:|:-----:|:------|
|requestCode| int|是|onActivityResult方法中对应的requestCode|
|data| Intent|是|onActivityResult方法中对应的data|
|selfRequestCode| int|是|和上面的自定义selfRequestCode对应|
|content|Context|是|上下文|
|LT-AppID|String|是|每个应用对应的appid|
|LTAppKey|String|是|每个应用对应的appKey|
|OnLoginSuccessListener|Interface|是|登录回调接口|

+ 3、接口说明

      /**
       * 登录回调
       */
       public interface OnLoginSuccessListener {
      /**
       * 成功回调
       *
       * @param result 成功信息
       */
      void onSuccess(BaseEntry<ResultData> result);
      
      /**
       * 请求失败
       *
       * @param ex 失败信息
       */
      void onFailed(Throwable ex);
      
      /**
       * 完成时
       */
      void onComplete();
      
      /**
       * 参数错误
       *
       * @param result 错误信息
       */
      void onParameterError(String result);
      
      /**
       * 错误回调
       *
       * @param error 错误信息
       */
      void onError(String error);
      
       }

  + 4、接口说明

         /**
          * 登录回调
          */
          public interface OnLoginSuccessListener {
         /**
          * 成功回调
          *
          * @param result 成功信息
          */
          void onSuccess(BaseEntry<ResultData> result);
          
         /**
          * 请求失败
          *
          * @param ex 失败信息
          */
         void onFailed(Throwable ex);
          
          /**
           * 完成时
           */
        void onComplete();

        /**
     ​    * 参数错误
     ​    *
     ​    * @param result 错误信息
     ​    */
     ​    void onParameterError(String result);

         /**
          * 错误回调
          *
          * @param error 错误信息
          */
         void onError(String error);

        }
     
   + 5、方法引用
        ​          
           1、登录方法
           GooglePlayLoginManager.initGoogle(this,clientID,REQUEST_CODE);
        ​       
           2、回调方法
        ​    GooglePlayLoginManager.onActivityResult(requestCode, data, REQUEST_CODE,
        ​    this,  LTAppID, LTAppKey,
        ​    new OnLoginSuccessListener() {
        ​        @Override
        ​        public void onSuccess(BaseEntry<ResultData> result) {
        ​            Log.e(TAG,result.getMsg()+"=="+result.getCode()+"=="+
        ​            result.getData().getApi_token());
        ​        }

                @Override
                public void onFailed(Throwable ex) {
                    Log.e(TAG,ex.getMessage());
                }
            
                @Override
                public void onComplete() {
            
                }
            
                @Override
                public void onParameterError(String result) {
                    Log.e(TAG,result);
                }
            
                @Override
                public void onError(String error) {
                    Log.e(TAG,error);
                }
            });

    + 6、结果码说明（具体可查看返回值）

                 "OK"或者200表示成功
                 "NO"表示登录失败
                 "12500"表示在google配置的sha1错误
    + 7、google登录配置
      
         [Google登录配置](https://blog.csdn.net/liutong123987/article/details/79417652)

  ### UI包配置说明

     1）、在项目的build下面引用仓库地址

         maven { url 'https://jitpack.io' }
  ![4](https://upload-images.jianshu.io/upload_images/1716569-84f44d0667d0283a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
     2）、在项目的app.build中引用UI的网络包如下所示：

       implementation 'com.github.muyishuangfeng:LTGameSdkUI:1.1.0'





+ 说明
  点击facebook开始facebook登录
  点击 google 开始 google登录 登录成功后跳转到协议页面，登录失败显示错误页面，之后再返回登录页面
 ![登录失败页面.png](https://upload-images.jianshu.io/upload_images/1716569-c12acecf7ff15284.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![协议页面.png](https://upload-images.jianshu.io/upload_images/1716569-aeab4b96bfb0c98e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


+ 协议传递方法说明
  
+ 1、登录方法

  |参数|类型|是否必须|说明|
  |:------:|:------:|:-----:|:------|
  |activity| Context|是|上下文|
  |mAgreementUrl| String|是|用户协议url|
  |mPrivacyUrl| String|是|隐私政策url|
  |googleClientID|String|是|google客户端ID|
  |LTAppID|String|是|乐推AppID|
  |LTAppKey|String|是|乐推AppKey|
  |OnReLoginInListener|Interface|是|第二次登录数据回调接口|

+ 2、登出方法
  |参数|类型|是否必须|说明|
  |:------:|:------:|:-----:|:------|
  |activity| Context|是|上下文|
  |mAgreementUrl| String|是|用户协议url|
  |mPrivacyUrl| String|是|隐私政策url|
  |googleClientID|String|是|google客户端ID|
  |LTAppID|String|是|乐推AppID|
  |LTAppKey|String|是|乐推AppKey|

+ 3、接口说明

      public interface OnReLoginInListener {
         //第二次登录成功结果回调
       void OnLoginResult(ResultData result);
       }

 
+ 3、使用说明

              //登录方法
              LoginUIManager.getInstance().loginIn(MainActivity.this, mAgreementUrl,
                      mPrivacyUrl, GoogleClientID, LTAppID, LTAppKey, new OnReLoginInListener() {
                          @Override
                          public void OnLoginResult(ResultData result) {

                          }
                      });
               //登出方法
               LoginUIManager.getInstance().loginOut(MainActivity.this, mAgreementUrl,
                      mPrivacyUrl, GoogleClientID, LTAppID, LTAppKey);

  
  
    
     
   + 使用方法
     1、在onCreate中注册

            EventUtils.register(this);
     2、在onStop或者onDestory方法中反注册

            EventUtils.unregister(this);
     3、接收方法
     

      @Subscribe
      public void receiveEvent(Event event) {
        switch (event.getCode()) {
            case BaseResult.MSG_RESULT_GOOGLE_SUCCESS: {//获取到信息后做登录等操作
                ResultData mData = (ResultData) event.getData();
                

                break;
            }
            
            case BaseResult.MSG_RESULT_FACEBOOK_SUCCESS: {//获取到信息后做登录等操作
                ResultData mData = (ResultData) event.getData();

                break;
            }
        }
    
    }

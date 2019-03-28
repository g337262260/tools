# 关于使用ShareSdk 进行第三方的登录遇到的问题

以下皆为在集成ShareSdk进行第三方的登录的时候走过的坑，希望能对看到的这篇文章的人有所帮助。

## QQ

- https://connect.qq.com
- 几乎没有坑，注册账号，等待审核，创建应用，获取Key

## Wechat

- https://open.weixin.qq.com

- 注册账号，微信开放平台开发者资质认证费用300，有效期一年

- 创建应用，获取key ,现在**AppSecret** 需要管理员用微信扫二维码之后才可以看，烦一批，看过之后妥善保存，如果遗忘，只能重置

- 在创建Android 应用时，应用签名为打包好的MD5签名,因为微信登录只能打包测试。

- 在集成sdk后，要在Manefest文件中注册WXEntryActivity，当初就是因为这个，做了半天无用功。

  ```xml
  <!-- 微信分享回调 -->
  <activity
      android:name=".wxapi.WXEntryActivity"
      android:configChanges="keyboardHidden|orientation|screenSize"
      android:exported="true"
      android:screenOrientation="portrait"
      android:theme="@android:style/Theme.Translucent.NoTitleBar" />
  ```

## 新浪微博

- http://open.weibo.com
- 几乎没有坑，注册账号，等待审核，创建应用，获取Key

## Facebook

- https://developers.facebook.com

- 注册账号

- 创建应用的时候需要Key Hashes,打包前后的值是不一样的，具体获取方法

  ```java
  try {
              PackageInfo info = getPackageManager().getPackageInfo( getPackageName(),  PackageManager.GET_SIGNATURES);
              for (Signature signature : info.signatures) {
                  MessageDigest md = MessageDigest.getInstance("SHA");
                  md.update(signature.toByteArray());
                  String KeyHash = Base64.encodeToString(md.digest(), Base64.DEFAULT);
                  Log.e("KeyHash:", "KeyHash:"+KeyHash);/
              }
          } catch (PackageManager.NameNotFoundException e) {
          } catch (NoSuchAlgorithmException e) {
          }
  ```

- 使用facebook  进行登录的时候不需要审核，但是分享需要审核，主要操作为：1.编写英文的分享步骤 2.录制分享的操作视频，便于审核人员进行测试（我还没有做分享，有不同意见可以指出）

## Twitter

- https://dev.twitter.com
- 使用海外手机号注册一个twitter账号（因为在shareSdk 官方文档上看到用海外手机号进行注册，所以也没有使用国内的。找了一个西班牙的，亲测可用）
- 创建应用，回调地址一定要和代码中的CallbackUrl 统一
- shareSdk 在集成twitter 登录的时候不会调用twitter 的app 进行授权，指挥在web端进行登录授权，就因为这个问题，纠结了半天，找了客服，他也没弄清楚，最后和ios的交流了一下才发现，调不起应用。

**注意 ：**

在测试过程中还出现了以下几个问题：

1. 因为翻墙的缘故，所以facebook 账号经常会发生账号异常，需要上传照片，审核过后才能接着使用
2. 在内部测试的时候，翻墙状态下是不能访问本地服务器的，所以登录接口总是不能使-.-!






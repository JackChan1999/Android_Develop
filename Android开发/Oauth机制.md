# 1. 登录
## **1.1 javaweb中如何去维持登录状态**

- 登录后信息放入 session中
- 页面内验证session中是否有登录信息
- 如果有，不需要再次登录
- 如果没有，跳转登录页面
- 如果登录后点击注销，删除session中登录信息，并清除页面缓存(必要的)

## **1.2 javaweb中哪些情况我们的session会过期**

- 过期-->很长时间没有去访问网站
- 主动关闭-->用户注销
- 切换浏览器

## **1.3 手机端如何维持登录状态**
通过sessionKey/tokenKey

登录协议

![登录协议](http://img.blog.csdn.net/20161112205830243)

![这里写图片描述](http://img.blog.csdn.net/20161112210024110)

###**1.3.1 sessionKey/tokenKey哪里来?**
登录成功之后,后台返回

###**1.3.2 sessionKey/tokenKey生成有什么规则?**
后台返回的，按照一定规则生成的(比如可以随机数生成一个24位以上的字符串)

###**1.3.3 登录成功返回的sessionkey/tokenKey存到哪里?**
保存到sp中就可以了

###**1.3.4 sessionkey/tokenKey使用场景**
有些协议需要用到登录信息，就需要看登录状态，就需要用到sessionkey/tokenKey，比如支付协议

![这里写图片描述](http://img.blog.csdn.net/20161112212056357)

###**1.3.5 如何使用sessionkey/tokenKey?**
需要登录状态，判断sp中是否有sessionkey/tokenKey?

- 有：那当前是已登录状态，就把个人信息和sessionkey/tokenKey上传到服务器;
- 没有：跳到登录界面，让用户登录

###**1.3.6 谁去判断sessionkey/tokenKey是否过期?**
任何协议把sessionkey/tokenKey传到服务器.服务器会判断sessionkey/tokenKey是否过期?

- 过期：告知客户端，登录状态已过期，需要重新登录
- 未过期：可以使用当前的登录信息，继续走逻辑
###**1.3.7 为什么判断是否过期需要后台做?**
因为前端可以修改当前时间?
###**1.3.8 sessionkey/tokenKey多久过期?**
这个具体看公司
###**1.3.9 后台如何去判断sessionkey/tokenKey是否过期？**
![这里写图片描述](http://img.blog.csdn.net/20161112212531631)

- 分配sessionkey/tokenKey的时候记录时间
- 判断是否存在对应的sessionKey：如果不存在，是不是直接就是无效.
- 某一时刻，用户判断sessionkey/tokenKey是否过期的时候，拿着当前时间和sessionkey/tokenKey分配时间做比较
- 大于指定时间：过期
- 没有大于指定时间：未过期

##2. logindemo
```java
public class MainActivity extends Activity {

	private Button	mLogin;
	private SPUtil	mSpUtil;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		mLogin = (Button) findViewById(R.id.login);
		mSpUtil = new SPUtil(MainActivity.this);
		mLogin.setOnClickListener(new OnClickListener() {

			@Override
			public void onClick(View v) {
				new Thread(new Runnable() {

					@Override
					public void run() {
						// TODO
						try {
							DefaultHttpClient httpClient = new DefaultHttpClient();
							HttpGet get = new HttpGet(
									"http://mobileif.maizuo.com/user/login?"
									+ "username=18682036558&"
									+ "password=96e79218965eb72c92a549dd5a330112&"
									+ "type=1&"
									+ "userType=1&"
									+ "appType=31&"
									+ "agentID=0-maizuo&"
									+ "loginType=1&"
									+ "clientID=31&"
									+ "channelID=31");
							HttpResponse response = httpClient.execute(get);

							if (response.getStatusLine().getStatusCode() == 200) {
								HttpEntity entity = response.getEntity();
								String result = EntityUtils.toString(entity);
								System.out.println("result :" + result);
								// 得到sessionKey保存到sp中
								JSONObject jsonObject = new JSONObject(result);
								String sessionKey = jsonObject.optString("sessionKey");

								// 保存到sp中
								mSpUtil.putString("sessionKey", sessionKey);
							}
						} catch (Exception e) {
							e.printStackTrace();
						}
					}
				}).start();
			}
		});

		findViewById(R.id.pay).setOnClickListener(new OnClickListener() {

			@Override
			public void onClick(View v) {
				// 这个地方需要使用sessionKey,对应的一般操作,首先判断是否存在
				String spSessionKey = mSpUtil.getString("sessionKey", "");
				if ("".equals(spSessionKey)) {
					Toast.makeText(getApplicationContext(), "请先登录", 0).show();
					return ;
				}
				//接着就是把对应的sessionKey和userid和payinfo上传给服务器
			}
		});
	}

}
```
返回数据

![sessionkey](http://img.blog.csdn.net/20161112213944849)
##3. 绘流程图说明sessionKey的使用
以确认支付协议为例

##验证码登录/注册_简单流程
1、提交手机号 2、下发短信验证码 3、提交验证码

![验证码登录](http://img.blog.csdn.net/20161112214934937)

验证码后台过期逻辑

![这里写图片描述](http://img.blog.csdn.net/20161112215320861)

##验证码登录/注册_流程描述

- 用户填写手机号，点击发送验证码，发送请求把手机号传到server
- server调用短信平台的接口知道发送内容，发送对象，完成短信的发送
- 用户收到短信，得到验证码，填写验证码，发送请求把手机号，验证码上传到server
- server判断我们的验证码是否正确，验证是否过期，根据不同的判断，返回不同的结果

##share sdk 短信验证码理解

![这里写图片描述](http://img.blog.csdn.net/20161112220218723)

#Android登录注册模块解决方案
原文链接：http://blog.csdn.net/wwj_748/article/details/50575642

几乎每个app都会有登录注册的功能，可以看看笔者开发的『南方周末新闻阅读器』，登录、手机注册、忘记密码这些入口，这些功能在app中要如何来实现呢？这个模块看似很简单，但要做好就需要考虑很多细节，比如对用户的输入的容错，操作的提示文案的设定，登录成功保存用户信息等等。

![登录注册](http://bbs.itheima.com/data/attachment/forum/201611/15/144919aczds5674cqdl4ml.png.thumb.jpg)

业务逻辑描述上一节的流程图已经很清晰的展现了登录注册的流程，这里继续用文字说明一下：
1. 点击进入个人中心或者需要用户登录状态的操作，先判断用户是否已经登录。
2. 如果已经登录，则继续后面的业务，否则，跳转到登录页面进行登录。
3. 如果已经有账号，则可以直接登录，或者可以直接选择第三方平台授权登录。
4. 如果未注册账号，则需要先进行账号注册，注册成功后再登录；也可以不注册账号，通过第三方平台授权进行登录。
5. 如果有账号，但忘记密码，则需要进行重置密码，否则直接登录。

具体实现登录可以使用账号登录，现在的app基本上都是手机号码登录，注册的时候也是一个手机对应一个账号，通过发送验证码进行验证；用户也可以选择第三方平台进行登录，一般会提供微信、QQ、新浪微博这样的主流社交平台进行授权登录，这里使用了友盟的SDK进行实现。

登录注册的解决方案，已经做成一个Demo，大家在实际开发的时候可以参考着根据自身的业务进行调整，但基本上不会差太多，第三方登录、验证码这个都可以选用第三方服务来实现

示例代码：LoginActivity.Java
```java
package com.devilwwj.loginandregister;  
   
import android.app.Activity;  
import android.content.Intent;  
import android.os.Bundle;  
import android.text.TextUtils;  
import android.text.method.HideReturnsTransformationMethod;  
import android.text.method.PasswordTransformationMethod;  
import android.view.KeyEvent;  
import android.view.View;  
import android.view.inputmethod.EditorInfo;  
import android.widget.TextView;  
import android.widget.Toast;  
   
import com.devilwwj.loginandregister.global.AppConstants;  
import com.devilwwj.loginandregister.utils.LogUtils;  
import com.devilwwj.loginandregister.utils.ProgressDialogUtils;  
import com.devilwwj.loginandregister.utils.RegexUtils;  
import com.devilwwj.loginandregister.utils.ShareUtils;  
import com.devilwwj.loginandregister.utils.SpUtils;  
import com.devilwwj.loginandregister.utils.ToastUtils;  
import com.devilwwj.loginandregister.utils.Utils;  
import com.devilwwj.loginandregister.views.CleanEditText;  
import com.umeng.socialize.bean.SHARE_MEDIA;  
import com.umeng.socialize.controller.UMServiceFactory;  
import com.umeng.socialize.controller.UMSocialService;  
import com.umeng.socialize.controller.listener.SocializeListeners.UMAuthListener;  
import com.umeng.socialize.controller.listener.SocializeListeners.UMDataListener;  
import com.umeng.socialize.exception.SocializeException;  
   
import java.util.Map;  
   
import static android.view.View.OnClickListener;  
   
/** 
 * @desc 登录界面 
 * Created by devilwwj on 16/1/24. 
 */ 
public class LoginActivity extends Activity implements OnClickListener {  
   
    private static final String TAG = "loginActivity";  
    private static final int REQUEST_CODE_TO_REGISTER = 0x001;  
   
    // 界面控件  
    private CleanEditText accountEdit;  
    private CleanEditText passwordEdit;  
   
    // 第三方平台获取的访问token，有效时间，uid  
    private String accessToken;  
    private String expires_in;  
    private String uid;  
    private String sns;  
   
    // 整个平台的Controller，负责管理整个SDK的配置、操作等处理  
    private UMSocialService mController = UMServiceFactory  
            .getUMSocialService(AppConstants.DESCRIPTOR);  
   
   
    @Override 
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_login);  
   
        initViews();  
   
        // 配置分享平台  
        ShareUtils.configPlatforms(this);  
    }  
   
    /** 
     * 初始化视图 
     */ 
    private void initViews() {  
        accountEdit = (CleanEditText) this.findViewById(R.id.et_email_phone);  
        accountEdit.setImeOptions(EditorInfo.IME_ACTION_NEXT);  
        accountEdit.setTransformationMethod(HideReturnsTransformationMethod  
                .getInstance());  
        passwordEdit = (CleanEditText) this.findViewById(R.id.et_password);  
        passwordEdit.setImeOptions(EditorInfo.IME_ACTION_DONE);  
        passwordEdit.setImeOptions(EditorInfo.IME_ACTION_GO);  
        passwordEdit.setTransformationMethod(PasswordTransformationMethod  
                .getInstance());  
        passwordEdit.setOnEditorActionListener(new TextView.OnEditorActionListener() {  
   
            @Override 
            public boolean onEditorAction(TextView v, int actionId,  
                                          KeyEvent event) {  
                if (actionId == EditorInfo.IME_ACTION_DONE  
                        || actionId == EditorInfo.IME_ACTION_GO) {  
                    clickLogin();  
                }  
                return false;  
            }  
        });  
    }  
   
    private void clickLogin() {  
        String account = accountEdit.getText().toString();  
        String password = passwordEdit.getText().toString();  
        if (checkInput(account, password)) {  
            // TODO: 请求服务器登录账号  
        }  
    }  
   
    /** 
     * 检查输入 
     * 
     * @param account 
     * @param password 
     * @return 
     */ 
    public boolean checkInput(String account, String password) {  
        // 账号为空时提示  
        if (account == null || account.trim().equals("")) {  
            Toast.makeText(this, R.string.tip_account_empty, Toast.LENGTH_LONG)  
                    .show();  
        } else {  
            // 账号不匹配手机号格式（11位数字且以1开头）  
            if ( !RegexUtils.checkMobile(account)) {  
                Toast.makeText(this, R.string.tip_account_regex_not_right,  
                        Toast.LENGTH_LONG).show();  
            } else if (password == null || password.trim().equals("")) {  
                Toast.makeText(this, R.string.tip_password_can_not_be_empty,  
                        Toast.LENGTH_LONG).show();  
            } else {  
                return true;  
            }  
        }  
   
        return false;  
    }  
   
    @Override 
    public void onClick(View v) {  
        Intent intent = null;  
        switch (v.getId()) {  
            case R.id.iv_cancel:  
                finish();  
                break;  
            case R.id.btn_login:  
                clickLogin();  
                break;  
            case R.id.iv_wechat:  
                clickLoginWexin();  
                break;  
            case R.id.iv_qq:  
                clickLoginQQ();  
                break;  
            case R.id.iv_sina:  
                loginThirdPlatform(SHARE_MEDIA.SINA);  
                break;  
            case R.id.tv_create_account:  
                enterRegister();  
                break;  
            case R.id.tv_forget_password:  
                enterForgetPwd();  
                break;  
            default:  
                break;  
        }  
    }  
   
    /** 
     * 点击使用QQ快速登录 
     */ 
    private void clickLoginQQ() {  
        if (!Utils.isQQClientAvailable(this)) {  
            ToastUtils.showShort(LoginActivity.this,  
                    getString(R.string.no_install_qq));  
        } else {  
            loginThirdPlatform(SHARE_MEDIA.QZONE);  
        }  
    }  
   
    /** 
     * 点击使用微信登录 
     */ 
    private void clickLoginWexin() {  
        if (!Utils.isWeixinAvilible(this)) {  
            ToastUtils.showShort(LoginActivity.this,  
                    getString(R.string.no_install_wechat));  
        } else {  
            loginThirdPlatform(SHARE_MEDIA.WEIXIN);  
        }  
    }  
   
    /** 
     * 跳转到忘记密码 
     */ 
    private void enterForgetPwd() {  
        Intent intent = new Intent(this, ForgetPasswordActivity.class);  
        startActivity(intent);  
    }  
   
    /** 
     * 跳转到注册页面 
     */ 
    private void enterRegister() {  
        Intent intent = new Intent(this, SignUpActivity.class);  
        startActivityForResult(intent, REQUEST_CODE_TO_REGISTER);  
    }  
   
   
    /** 
     * 授权。如果授权成功，则获取用户信息 
     * 
     * @param platform 
     */ 
    private void loginThirdPlatform(final SHARE_MEDIA platform) {  
        mController.doOauthVerify(LoginActivity.this, platform,  
                new UMAuthListener() {  
   
                    @Override 
                    public void onStart(SHARE_MEDIA platform) {  
                        LogUtils.i(TAG, "onStart------" 
                                + Thread.currentThread().getId());  
                        ProgressDialogUtils.getInstance().show(  
                                LoginActivity.this,  
                                getString(R.string.tip_begin_oauth));  
                    }  
   
                    @Override 
                    public void onError(SocializeException e,  
                                        SHARE_MEDIA platform) {  
                        LogUtils.i(TAG, "onError------" 
                                + Thread.currentThread().getId());  
                        ToastUtils.showShort(LoginActivity.this,  
                                getString(R.string.oauth_fail));  
                        ProgressDialogUtils.getInstance().dismiss();  
                    }  
   
                    @Override 
                    public void onComplete(Bundle value, SHARE_MEDIA platform) {  
                        LogUtils.i(TAG, "onComplete------" + value.toString());  
                        if (platform == SHARE_MEDIA.SINA) {  
                            accessToken = value.getString("access_key");  
                        } else {  
                            accessToken = value.getString("access_token");  
                        }  
                        expires_in = value.getString("expires_in");  
                        // 获取uid  
                        uid = value.getString(AppConstants.UID);  
                        if (value != null && !TextUtils.isEmpty(uid)) {  
                            // uid不为空，获取用户信息  
                            getUserInfo(platform);  
                        } else {  
                            ToastUtils.showShort(LoginActivity.this,  
                                    getString(R.string.oauth_fail));  
                        }  
                    }  
   
                    @Override 
                    public void onCancel(SHARE_MEDIA platform) {  
                        LogUtils.i(TAG, "onCancel------" 
                                + Thread.currentThread().getId());  
                        ToastUtils.showShort(LoginActivity.this,  
                                getString(R.string.oauth_cancle));  
                        ProgressDialogUtils.getInstance().dismiss();  
   
                    }  
                });  
    }  
   
   
    /** 
     * 获取用户信息 
     * 
     * @param platform 
     */ 
    private void getUserInfo(final SHARE_MEDIA platform) {  
        mController.getPlatformInfo(LoginActivity.this, platform,  
                new UMDataListener() {  
   
                    @Override 
                    public void onStart() {  
                        // 开始获取  
                        LogUtils.i("getUserInfo", "onStart------");  
                        ProgressDialogUtils.getInstance().dismiss();  
                        ProgressDialogUtils.getInstance().show(  
                                LoginActivity.this, "正在请求...");  
                    }  
   
                    @Override 
                    public void onComplete(int status, Map info) {  
   
                        try {  
                            String sns_id = "";  
                            String sns_avatar = "";  
                            String sns_loginname = "";  
                            if (info != null && info.size() != 0) {  
                                LogUtils.i("third login", info.toString());  
   
                                if (platform == SHARE_MEDIA.SINA) { // 新浪微博  
                                    sns = AppConstants.SINA;  
                                    if (info.get(AppConstants.UID) != null) {  
                                        sns_id = info.get(AppConstants.UID)  
                                                .toString();  
                                    }  
                                    if (info.get(AppConstants.PROFILE_IMAGE_URL) != null) {  
                                        sns_avatar = info  
                                                .get(AppConstants.PROFILE_IMAGE_URL)  
                                                .toString();  
                                    }  
                                    if (info.get(AppConstants.SCREEN_NAME) != null) {  
                                        sns_loginname = info.get(  
                                                AppConstants.SCREEN_NAME)  
                                                .toString();  
                                    }  
                                } else if (platform == SHARE_MEDIA.QZONE) { // QQ  
                                    sns = AppConstants.QQ;  
                                    if (info.get(AppConstants.UID) == null) {  
                                        ToastUtils  
                                                .showShort(  
                                                        LoginActivity.this,  
                                                        getString(R.string.oauth_fail));  
                                        return;  
                                    }  
                                    sns_id = info.get(AppConstants.UID)  
                                            .toString();  
                                    sns_avatar = info.get(  
                                            AppConstants.PROFILE_IMAGE_URL)  
                                            .toString();  
                                    sns_loginname = info.get(  
                                            AppConstants.SCREEN_NAME)  
                                            .toString();  
                                } else if (platform == SHARE_MEDIA.WEIXIN) { // 微信  
                                    sns = AppConstants.WECHAT;  
                                    sns_id = info.get(AppConstants.OPENID)  
                                            .toString();  
                                    sns_avatar = info.get(  
                                            AppConstants.HEADIMG_URL)  
                                            .toString();  
                                    sns_loginname = info.get(  
                                            AppConstants.NICKNAME).toString();  
                                }  
   
                                // 这里直接保存第三方返回来的用户信息  
                                SpUtils.putBoolean(LoginActivity.this,  
                                        AppConstants.THIRD_LOGIN, true);  
   
                                LogUtils.e("info", sns + "," + sns_id + "," 
                                        + sns_loginname);  
   
                                // TODO: 这里执行第三方连接(绑定服务器账号）  
   
   
                            }  
                        } catch (Exception e) {  
                            e.printStackTrace();  
                        }  
                    }  
   
                });  
    }  
}
```
示例代码：SignUpActivity.java

```java
package com.devilwwj.loginandregister;  
   
import android.app.Activity;  
import android.os.Bundle;  
import android.text.TextUtils;  
import android.util.Log;  
import android.view.KeyEvent;  
import android.view.View;  
import android.view.View.OnClickListener;  
import android.view.inputmethod.EditorInfo;  
import android.widget.Button;  
import android.widget.TextView;  
import android.widget.TextView.OnEditorActionListener;  
   
import com.devilwwj.loginandregister.utils.RegexUtils;  
import com.devilwwj.loginandregister.utils.ToastUtils;  
import com.devilwwj.loginandregister.utils.VerifyCodeManager;  
import com.devilwwj.loginandregister.views.CleanEditText;  
   
/** 
 * @desc 注册界面 
 * 功能描述：一般会使用手机登录，通过获取手机验证码，跟服务器交互完成注册 
 * Created by devilwwj on 16/1/24. 
 */ 
public class SignUpActivity extends Activity implements OnClickListener{  
    private static final String TAG = "SignupActivity";  
    // 界面控件  
    private CleanEditText phoneEdit;  
    private CleanEditText passwordEdit;  
    private CleanEditText verifyCodeEdit;  
    private Button getVerifiCodeButton;  
   
    private VerifyCodeManager codeManager;  
   
    @Override 
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_signup);  
   
        initViews();  
        codeManager = new VerifyCodeManager(this, phoneEdit, getVerifiCodeButton);  
   
    }  
   
    /** 
     * 通用findViewById,减少重复的类型转换 
     * 
     * @param id 
     * @return 
     */ 
    @SuppressWarnings("unchecked")  
    public final  E getView(int id) {  
        try {  
            return (E) findViewById(id);  
        } catch (ClassCastException ex) {  
            Log.e(TAG, "Could not cast View to concrete class.", ex);  
            throw ex;  
        }  
    }  
   
   
    private void initViews() {  
   
        getVerifiCodeButton = getView(R.id.btn_send_verifi_code);  
        getVerifiCodeButton.setOnClickListener(this);  
        phoneEdit = getView(R.id.et_phone);  
        phoneEdit.setImeOptions(EditorInfo.IME_ACTION_NEXT);// 下一步  
        verifyCodeEdit = getView(R.id.et_verifiCode);  
        verifyCodeEdit.setImeOptions(EditorInfo.IME_ACTION_NEXT);// 下一步  
        passwordEdit = getView(R.id.et_password);  
        passwordEdit.setImeOptions(EditorInfo.IME_ACTION_DONE);  
        passwordEdit.setImeOptions(EditorInfo.IME_ACTION_GO);  
        passwordEdit.setOnEditorActionListener(new OnEditorActionListener() {  
   
            @Override 
            public boolean onEditorAction(TextView v, int actionId,  
                                          KeyEvent event) {  
                // 点击虚拟键盘的done  
                if (actionId == EditorInfo.IME_ACTION_DONE  
                        || actionId == EditorInfo.IME_ACTION_GO) {  
                    commit();  
                }  
                return false;  
            }  
        });  
    }  
   
    private void commit() {  
        String phone = phoneEdit.getText().toString().trim();  
        String password = passwordEdit.getText().toString().trim();  
        String code = verifyCodeEdit.getText().toString().trim();  
   
        if (checkInput(phone, password, code)) {  
            // TODO:请求服务端注册账号  
        }  
    }  
   
    private boolean checkInput(String phone, String password, String code) {  
        if (TextUtils.isEmpty(phone)) { // 电话号码为空  
            ToastUtils.showShort(this, R.string.tip_phone_can_not_be_empty);  
        } else {  
            if (!RegexUtils.checkMobile(phone)) { // 电话号码格式有误  
                ToastUtils.showShort(this, R.string.tip_phone_regex_not_right);  
            } else if (TextUtils.isEmpty(code)) { // 验证码不正确  
                ToastUtils.showShort(this, R.string.tip_please_input_code);  
            } else if (password.length() < 6 || password.length() > 32 
                    || TextUtils.isEmpty(password)) { // 密码格式  
                ToastUtils.showShort(this,  
                        R.string.tip_please_input_6_32_password);  
            } else {  
                return true;  
            }  
        }  
   
        return false;  
    }  
   
    @Override 
    public void onClick(View v) {  
        switch (v.getId()) {  
            case R.id.btn_send_verifi_code:  
                // TODO 请求接口发送验证码  
                codeManager.getVerifyCode(VerifyCodeManager.REGISTER);  
                break;  
   
            default:  
                break;  
        }  
    }  
}
```

#QQ登录

网址：http://wiki.connect.qq.com/sdk%E4%B8%8B%E8%BD%BD
集成步骤(移动应用接入流程)

1.注册成为开发者，登录
2.点击申请加入，创建应用-->会分配appId，appKey

- APP ID：1104072093
- APP KEY：PzMWIM4GYZvxGRjd

3.完善信息
4.下载demo.运行看效果
5.集成开发

- webview方式(老方式)：授权的时候是跳到了一个webview上去授权
- sso方式：如果手机里面装了qq.那就是单点登录的形式.如果没有装qq，就是跳到webview这种老方式;
- sso：单点登录

集成步骤_官方

1.注册成为开发者
2.创建应用
3.完善应用信息
4.提交上线申请
5.通过审核上线

![这里写图片描述](http://img.blog.csdn.net/20161113000257011)

![这里写图片描述](http://img.blog.csdn.net/20161113000133774)

通过广播的方式拿到access_token

#新浪微博
![oauth](http://img.blog.csdn.net/20161113114224944)

1、点击了发起授权请求的按钮，使用WebView加载登陆网页，也是oauth2.0授权步骤中的第一步，其实就是去打开了一个授权的网页

通过Get方式传递三个参数: client_id(应用的appkey)，redirect_uri(回调地址)，display(展示方式，手机设备为mobile)

https://auth.sina.com.cn/oauth2/authorize?client_id=3237555059&redirect_uri=http%3A%2F%2Fbilly.itheima.com&display=mobile

2、用户授权完成之后，回调到回调页，同时传递code(授权码)

在wap页点击登陆按钮后, 会有一个回调地址, 可以在WebViewClient的shouldOverrideUrlLoading方法中捕获。

用户点击登录后, 是一个302跳转, 跳转地址就是申请appkey时填写的Redirect URL, 登录成功后这个地址后面带有code参数. 如果是移动端的话, 这个地址不需要有实体的页面

回调页：http://billy.itheima.com/?code=ce97366666b1b902575ce207401477ff&state=

3、拿着授权的code请求access_token

请求地址:https://auth.sina.com.cn/oauth2/access_token
请求方式:post
请求参数的形式:key-value
请求具体参数
client_id=3237555059&client_secret=2b6c964b071e2ecc28c1835628cc6901&grant_type=authorization_code&code=ce97366666b1b902575ce207401477ff&redirect_uri=http%3A%2F%2Fbilly.itheima.com

```
{
       "access_token": "26883566623yk3e3x6rXZ3gOj5J35b6d",
       "expires_in": 1437982314,
       "time_left": 22868,
       "uid": "1884774904",
       "refresh_token": "f244f166623yk3e3x6rXZ3gOj5Jf1bb7"
 }
```
获取用户信息

https://api.weipan.cn/2/account/info?access_token=26883566623yk3e3x6rXZ3gOj5J35b6d

![oauth](http://img.blog.csdn.net/20161113113626550)

![oauth](http://img.blog.csdn.net/20161113112921498)

![oauth](http://img.blog.csdn.net/20161113112931889)

##**access_token加密**
access_token的保存，有关加密算法的相关知识，[请看Android安全加密系列文章](http://blog.csdn.net/axi295309066/article/details/52491077)

![token保存](http://img.blog.csdn.net/20161113114515253)

![token保存](http://img.blog.csdn.net/20161113114526039)

密码规则

```
this.PASSWORD = "com.sina.vdisk.security.password.d7af3082d815945ff47ae58647bd9436"
                + IMEI + appKeyPair.key + appKeyPair.secret;
```

![密码规则](http://img.blog.csdn.net/20161113115145182)

##**IMEI**
有别于sim卡的序列号，可以作为手机的唯一标识，类似我们pc的机器码

两个概念

- imei：设备的唯一标识
- imsi：移动sim卡的唯一标识

买手机：3码合一 手机序列号 电池序列号 手机包装序列号

统计apk的安装量：启动应用程序的时候，把手机的imei号上传到服务器

![统计apk安装量](http://img.blog.csdn.net/20161113120821987)

获取IMEI

```java
TelephonyManager telephonyManager = (TelephonyManager) ctx .getSystemService(Context.TELEPHONY_SERVICE);
telephonyManager.getDeviceId();
```
##**3层加密**
只是加大了被破解的难度

- des加密-->密码唯一化，复杂化(但是还是不安全)

```
this.PASSWORD = "com.sina.vdisk.security.password.d7af3082d815945ff47ae58647bd9436" + IMEI + appKeyPair.key + appKeyPair.secret;
```

- 秘钥放到so库里面，通过jni调用->密码放到so库里面，这个时候，加到了获取秘钥的难度。(同样，反编译apk，可以拿到so库，然后可以调用本地方法获取到密码)

```java
	// 本地方法 c实现
	public native String getDataFromC();

	//从c中获取秘钥
	public static String getKey(){
		return getInstance().getDataFromC();
	}
```

```
#include <jni.h>

jstring Java_com_example_level3encrypt_util_LanguageTransform_getDataFromC(JNIEnv* env,
		jobject obj) {
	//JNIEnv* env  jni环境的上下文对象
	//jobject obj  调用该方法的java对象

	//返回一个java字符串,即秘钥
	char* cstr = "myKey0510YUY1234";

	//将c字符串 转换成 java字符串
	//	   jstring     (*NewStringUTF)(JNIEnv*, const char*);
	jstring jstr = (**env).NewStringUTF(env, cstr);
	return jstr;
}

```

- 混淆(也是可以拿到，混淆的时候。我们的字符串是不会进行混淆的。只是混淆我们方法名，以及变量名)

#第三方登录
- 用qq号/微信号/微博号去登录自己的应用
- 核心：就是拿到我们access_token
- 实际开发第三方登录的协议：http://xxx?token=>xxx&type=x，token就是我们授权之后返回的access_token，type是为了区分不同登录渠道

第三方登录协议

![第三方登录协议](http://img.blog.csdn.net/20161120021115253)

#access_token是啥，干嘛用?

形象解释：申请调兵-->皇帝同意-->兵符-->开始调兵
拿到用户在第三方平台的唯一的标识;
获取用户的nickname，头像，邮箱等其他信息;

##实际开发3大步
app做的事情，实际开发，我们能把我这里的几个步骤，就可以完成开发工作

1.发起授权请求，让用户授权（输入账号密码）
2.处理授权结果，拿到access_token
3.调用第三方登录协议(自己公司定义)，传递access_token到app的server，后续逻辑交给server 

##server拿到access_token做了什么
![这里写图片描述](http://img.blog.csdn.net/20161113010631295)

静默注册新用户或者返回已有用户用户信息

1.使用access_token拿到用户在第三方平台的唯一ID;
判断第三方平台的唯一ID是否存在我们的用户信息表中;
存在：(之前使用qq号登陆过自己的系统)返回当前用户的用户信息
不存在：(用户还没有使用过此qq登陆过我们的系统)
2. 调用相关的接口，拿到nickname，邮箱，头像(需要什么拿取什么);

##qq登录_sso形式

sso(单点登录)方式：如果手机里面装了qq，那就是单点登录的形式，如果没有装qq，就是跳到webview这种老方式

返回参数

![这里写图片描述](http://img.blog.csdn.net/20161113101753778)

#**什么是开发平台**
![这里写图片描述](http://img.blog.csdn.net/20161112234000759)

- 开放平台（Open Platform） 在软件业和网络中，开放平台是指软件系统通过公开其应用程序编程接口（API）或函数（function)来使外部的程序可以增加该软件系统的功能或使用该软件系统的资源，而不需要更改该软件系统的源代码
- 开发平台的作用：提供功能，提供资源

#**OAuth**
OAuth是一个开放标准，允许用户让第三方应用访问该用户在某一网站上存储的私密的资源（如照片，视频，联系人列表），而无需将用户名和密码提供给第三方应用

##**oauth产生背景**
传统互联网时代，各个网站和服务之间是封闭的，数据无法进行交互。随着互联网技术的迅猛发展、系统之间的相互协作日益增多，开放与共享数据的需求不断增加，互联网服务之间的整合已经成为必然趋势，一个通过自身开放平台来实现数据互通甚至用户共享的时代已经来临。把网站的服务封装成一系列计算机易识别的数据接口开放出去，供第三方开发者使用，这种行为就叫做Open API，提供开放API 的平台本身就被称为开放平台。

2007 年5 月份，Facebook 宣布改版，最早提出了开放平台的概念，正式从一个社交网站向一个社交应用平台转型。至此，开放平台发展迅速，成为互联网发展的趋势，也是一种革命性的发展模式

国内外已经有很多的公司开发了自己的开放平台，Facebook、Twitter、腾讯、新浪微博、人人网，它们用户规模大、技术实力强，为第三方提供了一整套自成体系、纷繁复杂的“开放API”。通过开放平台，网站不仅能提供对Web 网页的简单访问，还可以进行复杂的数据交互，允许第三方开发者利用其资源开发复杂的应用，既丰富自身网站应用，为用户提供更好的服务，逐步建立起一个服务完备的网络社会，也为第三方连接的网站带来更多的用户。开放平台迅速成为互联网发展的趋势。

开放平台的核心问题在于用户验证和授权。对于服务提供商来说，一般不会希望第三方直接使用用户名和密码来验证用户身份，除非双方具有很强的信任关系。OAuth 协议正是为了解决服务整合时“验证和授权”这一根本问题而产生的，具有同样认证功能的协议还有Openid。

![oauth](http://img.blog.csdn.net/20161112153348478)

如果一个用户拥有两项服务：一项服务是图片在线存储服务A，另一个是图片在线打印服务B。如下图所示。由于服务A与服务B是由两家不同的服务提供商提供的，所以用户在这两家服务提供商的网站上各自注册了两个用户，假设这两个用户名各不相同，密码也各不相同。当用户要使用服务B打印存储在服务A上的图片时，用户该如何处理？

方法一：用户可能先将待打印的图片从服务A上下载下来并上传到服务B上打印，这种方式安全但处理比较繁琐，效率低下

方法二：用户将在服务A上注册的用户名与密码提供给服务B，服务B使用用户的帐号再去服务A处下载待打印的图片，这种方式效率是提高了，但是安全性大大降低了，服务B可以使用用户的用户名与密码去服务A上查看甚至篡改用户的资源。很多公司和个人都尝试解决这类问题，包括Google、Yahoo、Microsoft，这也促使OAUTH项目组的产生。OAuth是由Blaine Cook、Chris　　Messina、Larry Halff 及David Recordon共同发起的，目的在于为API访问授权提供一个开放的标准。

##**什么是OAuth**
![这里写图片描述](http://img.blog.csdn.net/20161112234430113)

OAuth是一个开放标准，允许用户让第三方应用访问该用户在某一网站上存储的私密的资源（如照片，视频，联系人列表），而无需将用户名和密码提供给第三方应用。

OAuth允许用户提供一个令牌，而不是用户名和密码来访问他们存放在特定服务提供者的数据。每一个令牌授权一个特定的网站（例如，视频编辑网站)在特定的时段（例如，接下来的2小时内）内访问特定的资源（例如仅仅是某一相册中的视频）。这样，OAuth允许用户授权第三方网站访问他们存储在另外的服务提供者上的信息。

简单：不管是OAUTH服务提供者还是应用开发者，都很容易于理解与使用
安全：没有涉及到用户密钥等信息，更安全更灵活
开放：任何服务提供商都可以实现OAUTH，任何软 件开发商都可以使用OAUTH

##**OAuth的核心工作流程**

OAuth 为客户端提供了一种代表资源拥有者访问受保护资源的方法。在客户端访问受保护资源之前，它必须先从资源拥有者获取授权（访问许可），然后用访问许可交换访问令牌（Access Token，包含许可的作用域、持续时间和其它属性等信息）。客户端通过向资源服务器出示访问令牌来访问受保护资源。

###**OAuth1.0**
OAuth1.0主要流程是四步骤：

1、获取未授权的Request Token
2、请求用户授权Request Token
3、使用授权后的Request Token换取Access Token
4、使用 Access Token 访问或修改受保护资源。

![这里写图片描述](http://img.blog.csdn.net/20161112154430044)

1.0的缺陷：

1、oauth1.0对手机客户端，移动设备等非server第三方的支持不好。是因为oauth1.0将多种流程合并成了一种，而事实证明，这种合并的流程体验性非常差。
2、oauth1.0的三步认证过程比较繁琐和复杂，对第三方开发者增加了极大的开发难度。
3、oauth1.0的加密需求过于复杂，第三方开发者使用oauth之前需要花费精力先实现oauth1.0的加密算法。
4、oauth1.0生成的access_token要求是永久有效的，这导致的问题是网站的安全性和破坏网站的架构。

###**OAuth2.0**

oauth2.0针对1.0的各种问题提供了解决方法：

1、 oauth2.0提出了多种流程，各个客户端按照实际情况选择不同的流程来获取access_token。这样就解决了对移动设备等第三方的支持，也解决了拓展性的问题 

2 、oauth2.0删除了繁琐的加密算法。利用了https传输对认证的安全性进行了保证 

3、 oauth2.0的认证流程一般只有2步，对开发者来说，减轻了负担 

4 、oauth2.0提出了access_token的更新方案，获取access_token的同时也获取refresh_token， access_token是有过期时间的，refresh_token的过期时间较长，这样能随时使用refresh_token对access_token进行更新

![这里写图片描述](http://img.blog.csdn.net/20161112181823627)

**oauth2.0授权流程**

- 用户打开客户端以后，客户端要求用户给予授权。
- 用户同意给予客户端授权。
- 客户端使用上一步获得的授权，向认证服务器(比如qq登录，那就是腾讯)申请令牌。
- 认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
- 客户端使用令牌，向资源服务器申请获取资源。
- 资源服务器确认令牌无误，同意向客户端开放资源

授权流程图

![授权流程图](http://img.blog.csdn.net/20161112164809915)

Oauth2.0根据授权方式和客户端状态的不同对应不一样的实现方式。

##**Webserver方式**

适用场景：Web Server子态适用于有能力与终端用户的user-agent（通常是浏览器）交互并能够从授权服务器接收（通过重定向）请求（即有能力担当HTTP服务器的角色）的客户端。

![这里写图片描述](http://img.blog.csdn.net/20161112164920996)

##**userAgent方式**

适用场景：Web Server子态适用于有能力与终端用户的user-agent（通常是浏览器）交互并能够从授权服务器接收（通过重定向）请求（即有能力担当HTTP服务器的角色）的客户端 ，典型的例子是用诸如JavaScript语言编写并运行在浏览器的程序。这些客户端不能保存客户端私有证书，并且客户端的验证基于user-agent的同源策略。

说明：在其它子态中，客户端对于终端用户的授权和访问令牌使用分开的不同请求来完成，而与之不同的是，在user-agent子态中，客户端以HTTP重定向的方式在终端用户授权请求的结果中获取到访问令牌。客户端请求授权服务器将user-agent重定向到另一个web服务器或user-agent能访问到的本地资源，而且user-agent有能力从响应信息中提取出访问令牌并传给客户端。

![这里写图片描述](http://img.blog.csdn.net/20161112165003212)

##原生程序方式

适用场景：原生程序方式适用于作为原生代码运行在终端用户计算机或移动设备上的客户端。这些客户端通常有能力与终端用户的user-agent交互（或嵌入user-agent）。


基于不同的需求和期望的终端用户体验，原生程序客户端可以有三种方式实现：

外部启用一个user-agent；
内嵌一个user-agent；
直接要求用户输入用户名和密码。

##优缺点：
外部浏览器可能会提高完成效率，因为终端用户可能已经登录过了而不需要重新进行身份验证。

嵌入的user-agent通常能提供更好的用户流程，因为它不必切换上下文并打开新窗口。

但嵌入的user-agent对安全提出了挑战，因为用户在一个无法辨别的窗口之中进行身份验证，而不像很多user-agent那样能提供可视化的保护。

第三种方式，实现起来最简单，但它将终端用户的密码直接交给了第三方客户端，而客户端不得不用明文存储密码，所以它要求客户端和认证服务方有很强的信任关系。

![这里写图片描述](http://img.blog.csdn.net/20161112165110041)

##刷新令牌

OAuth2.0引入刷新令牌的方式重新获取访问令牌。访问令牌的生命周期通常比资源拥有者授予的要短一些。当分发一个访问令牌时，授权服务器可以同时传回一个刷新令牌，在当前访问令牌超时后，客户端可以用这个刷新令牌重新获取一个访问令牌。当请求新的访问令牌时，刷新令牌担当起访问许可的角色。使用刷新令牌，不再需要再次与资源拥有者交互，也不需要存储原始的访问许可来获得访问令牌和刷新令牌。

注：可以使用刷新令牌也可以直接使用客户端的私有证书

![这里写图片描述](http://img.blog.csdn.net/20161112165143839)

##**oauth2.0涉及的角色**

-  **Third-party application**：第三方应用程序，本文中又称"客户端"（client），即上一节例子中的"云冲印"
-  **HTTP service**：HTTP服务提供商，本文中简称"服务提供商"，即上一节例子中的Google
-  **Resource Owner**：资源所有者，本文中又称"用户"（user）。
-  **User Agent**：用户代理，本文中就是指浏览器
-  **Authorization server**：认证服务器，即服务提供商专门用来处理认证的服务器
-  **Resource server**：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器

说明：本文提出的OAuth使用方式是基于2.0协议，只对OAuth2.0 最核心的工作流程进行设计，即获取Access Token 的过程。

![这里写图片描述](http://img.blog.csdn.net/20161112155342968)

#服务器端的数据存储方案

OAuth2.0 机制的核心工作流程，大部分信息，如APP 信息、用户和APP 之间的授权关系、Access Token等，需要永久性存储，存储在数据库中。一些不需要永久存储的信息，如临时授权码，只使用一次，而且，在很短的时间内就要使其失效，就没有必要存储在数据库之中，可存储在譬如memcached 等缓存系统中，即提高了系统的处理速度，又减少了数据库压力

#数据库表结构设计

客户端要先注册一个应用，获取该应用的APPID和APPSECRET，应用的详细信息存储在数据表中，如下图所示：

![这里写图片描述](http://img.blog.csdn.net/20161112154835609)

用户授权信息表用来保存授权成功的访问令牌。

![这里写图片描述](http://img.blog.csdn.net/20161112154907806)

![这里写图片描述](http://img.blog.csdn.net/20161112154933984)

#**OAuth和Openid的比较**

##**Openid是什么**

OpenID 是一个以用户为中心的数字身份识别框架。

OpenID 的创建基于这样一个概念：我们可以通过 URI （又叫 URL 或网站地址）来认证一个网站的唯一身份，同理，我们也可以通过这种方式来作为用户的身份认证。由于URI 是整个网络世界的核心，它为基于URI的用户身份认证提供了广泛的、坚实的基础。

OAuth和Openid同为认证机制，但它们的侧重点不一样。

OAuth关注的授权，即：“用户能做什么” ；
而OpenID关注的是证明，即：“用户是谁”。

下面就分别来举例说明两者的功能。

###**OpenID**

- 用户希望访问其在example.com的账户
- example.com 提示用户输入他/她/它的OpenID
- 用户给出了他的OpenID，比如说”http://user.myopenid.com”
- example.com 跳转到了用户的OpenID提供商“mypopenid.com”
- 用户在”myopenid.com”(OpenID provider)提示的界面上输入用户名密码登录
- “myopenid.com” (OpenID provider) 问用户是否要登录到example.com
- 用户同意后，”myopenid.com” (OpenID provider) 跳转回example.com
- example.com 允许用户访问其资源

###**OAuth**

- 用户在使用example.com时希望从mycontacts.com导入他的联系人
- example.com (在OAuth的黑话里面叫“Consumer”)把用户送往mycontacts.com (黑话是“Service Provider”)
- 用户在mycontacts.com 登录
- mycontacts.com问用户是不是希望授权example.com访问他在mycontact.com的联系人
- 用户确定
- mycontacts.com 把用户送回example.com
- example.com 从mycontacts.com拿到联系人
- example.com 告诉用户导入成功







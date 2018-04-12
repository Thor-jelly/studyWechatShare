# 微信开放平台-分享到你的朋友圈
> [学习慕课网](https://www.imooc.com/learn/455)

# 注册微信开放平台账号
到该网站注册账号-->>[微信·开放平台](https://open.weixin.qq.com)  
**注册的邮箱一定不能跟微信绑定，如果绑定就换个邮箱就好了**  

# 创建你的应用在开发平台
在管理中心创建你的移动应用，到应用签名使用[android资源下载](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419319167&token=d94c2ce9a3ea504cec90c14df6e0c3c047abd044&lang=zh_CN)中的签名工具获取就好了。  
可以参考[android集成微信分享](https://www.jianshu.com/p/e1a3f6844079)

# 开发流程

```
public class MainActivity extends AppCompatActivity {

    private TextView mResultTv;
    private Button mRegBtn;
    private Button mGotoSendBtn;
    private Button mLaunchWxBtn;
    private Button mCheckTimelineSupportedBtn;

    /**
     * 微信程序中的ID
     */
    private static final String APP_ID = "";
    public static IWXAPI mIWXAPI;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        regToWx();
        assignViews();
        initEvent();
    }

    private void initEvent() {
//        注册
        mRegBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

            }
        });
//                调准到发送消息
        mGotoSendBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //向好友或朋友圈发送文本
                sendTextStr("我发送了一个分享。。。", System.currentTimeMillis() + "");
            }
        });
//        启动微信
        mLaunchWxBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "是否安装微信-->>" + mIWXAPI.isWXAppInstalled(), Toast.LENGTH_SHORT).show();
                Toast.makeText(MainActivity.this, "是否打开微信-->>" + mIWXAPI.openWXApp(), Toast.LENGTH_SHORT).show();
            }
        });
//                检查是否支持朋友群
        mCheckTimelineSupportedBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "是否支持朋友圈-->>" + mIWXAPI.isWXAppSupportAPI(), Toast.LENGTH_SHORT).show();
                Toast.makeText(MainActivity.this, "是否支持朋友圈111-->>" + mIWXAPI.getWXAppSupportAPI(), Toast.LENGTH_SHORT).show();
                //Build.PAY_SUPPORTED_SDK_INT大于于mIWXAPI.getWXAppSupportAPI()就表示不支持微信支付
            }
        });
    }

    /**
     * 向好友或朋友圈发送文本
     *
     * @param sendStr     发送的信息
     * @param transaction 跟微信交互对象的唯一标示
     */
    private void sendTextStr(String sendStr, String transaction) {
        //1.初始化创建分享文本对象
        WXTextObject wxTextObject = new WXTextObject();
        wxTextObject.text = sendStr;

        //2.创建传输对象 用于android向微信发送数据
        WXMediaMessage wxMediaMessage = new WXMediaMessage();
        wxMediaMessage.mediaObject = wxTextObject;
        //设置一个描述
        wxMediaMessage.description = "我是分享描述";

        //3.创建跟微信交互的对象
        SendMessageToWX.Req req = new SendMessageToWX.Req();
        req.message = wxMediaMessage;
        //唯一标示
        req.transaction = transaction;
        //场景-->>表示发送给朋友(SendMessageToWX.Req.WXSceneSession)
        //             还是朋友圈(SendMessageToWX.Req.WXSceneTimeline)
        req.scene = SendMessageToWX.Req.WXSceneTimeline;

        //4.发送给微信客户端 发送成功返回true 失败返回false
        boolean sendReq = mIWXAPI.sendReq(req);
        Toast.makeText(this, "发送微信客户端是否成功：" + sendReq, Toast.LENGTH_SHORT).show();
    }

    private void assignViews() {
        mResultTv = (TextView) findViewById(R.id.result_tv);
        mRegBtn = (Button) findViewById(R.id.reg_btn);
        mGotoSendBtn = (Button) findViewById(R.id.goto_send_btn);
        mLaunchWxBtn = (Button) findViewById(R.id.launch_wx_btn);
        mCheckTimelineSupportedBtn = (Button) findViewById(R.id.check_timeline_supported_btn);
    }

    private void regToWx() {
        mIWXAPI = WXAPIFactory.createWXAPI(this, APP_ID, true);
        mIWXAPI.registerApp(APP_ID);
    }
}
```


```
<activity
            android:name=".wxapi.WXEntryActivity"
            android:launchMode="singleTask"
            android:exported="true" />
```

<b>在注册的包名下面创建一个叫"wxapi"的包  
package com.zhouyijin.zyj.fakeshanbay.wxapi;  
创建指定名字的Activity  
创建一个Acitivty,必须是这个名字!  
public class WXEntryActivity extends AppCompatActivity {}  </b>

```
/**
 * 类描述：//TODO:(这里用一句话描述这个方法的作用)    <br/>
 * 创建人：吴冬冬<br/>
 * 创建时间：2018/4/11 16:45 <br/>
 */
public class WXEntryActivity extends AppCompatActivity implements IWXAPIEventHandler {

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.entry);

        //如果分享的时候，该界面没有开启，那么微信开始这个activity时，会调用onCreate，所以这里要处理微信的返回结果
        MainActivity.mIWXAPI.handleIntent(getIntent(), this);
    }

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);

        //如果分享的时候，该已经开启，那么微信开始这个activity时，会调用onNewIntent，所以这里要处理微信的返回结果
        setIntent(intent);
        MainActivity.mIWXAPI.handleIntent(getIntent(), this);
    }

    @Override
    public void onResp(BaseResp resp) { //在这个方法中处理微信传回的数据
        Toast.makeText(this, "errCode->"+resp.errCode, Toast.LENGTH_SHORT).show();
        //形参resp 有下面两个个属性比较重要
        //1.resp.errCode
        //2.resp.transaction则是在分享数据的时候手动指定的字符创,用来分辨是那次分享(参照4.中req.transaction)
        switch (resp.errCode) { //根据需要的情况进行处理
            case BaseResp.ErrCode.ERR_OK:
                //正确返回
                Toast.makeText(this, "正确返回", Toast.LENGTH_SHORT).show();
                break;
            case BaseResp.ErrCode.ERR_USER_CANCEL:
                //用户取消
                Toast.makeText(this, "用户取消", Toast.LENGTH_SHORT).show();
                break;
            case BaseResp.ErrCode.ERR_AUTH_DENIED:
                //认证被否决
                Toast.makeText(this, "认证被否决", Toast.LENGTH_SHORT).show();
                break;
            case BaseResp.ErrCode.ERR_SENT_FAILED:
                //发送失败
                Toast.makeText(this, "发送失败", Toast.LENGTH_SHORT).show();
                break;
            case BaseResp.ErrCode.ERR_UNSUPPORT:
                //不支持错误
                Toast.makeText(this, "不支持错误", Toast.LENGTH_SHORT).show();
                break;
            case BaseResp.ErrCode.ERR_COMM:
                //一般错误
                Toast.makeText(this, "一般错误", Toast.LENGTH_SHORT).show();
                break;
            default:
                //其他不可名状的情况
                Toast.makeText(this, "其他不可名状的情况->"+resp.errCode, Toast.LENGTH_SHORT).show();
                break;
        }
    }

    @Override
    public void onReq(BaseReq req) {
        //......这里是用来处理接收的请求,暂不做讨论
    }

}
```

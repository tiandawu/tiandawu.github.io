---
title: QQ抢红包插件实现
date: 2016-01-15 10:43:11
tags: Android
comments: true
categories: Android
description: 之前跟那些20年的单身狗抢红包是屡战屡败。再加上最近群里面发红包发的厉害，又想到快要过年了，到时候还不知道群里要发好多红包，所以我将之前在网上宕的一份微信抢红包的代码修改了一下，实现了QQ抢红包！可以支持抢QQ拼手气红包，普通红包，口令红包，现在再也不怕20年单身手速`的人跟我抢红包了！
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之前跟那些20年的单身狗抢红包是屡战屡败。再加上最近群里面发红包发的厉害，又想到快要过年了，到时候还不知道群里要发好多红包，所以我将之前在网上宕的一份微信抢红包的代码修改了一下，实现了QQ抢红包！可以支持抢QQ**拼手气红包**，**普通红包**，**口令红包**，现在再也不怕**20年单身手速**的人跟我抢红包了！
### 先看测试效果图：
##### 1. 抢QQ**口令红包：**    
![口令红包.gif](http://upload-images.jianshu.io/upload_images/1440183-6788c271c29e0d39.gif?imageMogr2/auto-orient/strip)  

可以看见，只要红包一发出，自动填写口令并发出，帮你将红包抢到手！
##### 2. 抢QQ**拼手气红包：**
![拼手气红包.gif](http://upload-images.jianshu.io/upload_images/1440183-f6a3dc7fabbd2d3b.gif?imageMogr2/auto-orient/strip)  

拼手气红包也是一样，只要红包一发出，自动帮你把红包抢到手，是不是很爽的感觉？  
##### 3. 抢QQ好友发送的**红包：**  
![好友发送的QQ口令红包.gif](http://upload-images.jianshu.io/upload_images/1440183-936802d3cea53f84.gif?imageMogr2/auto-orient/strip)  

只要好友或者群里的人把红包一发出，就会第一时间让你抢到红包！
所以只要在群里面开启插件，抢红包从来都是百发百中！  

好了废话不多说了，也不吹嘘有多牛多好了，下面直接给大家上**代码：**


**MainActivity:**

```java
	/*MainActivity中的代码基本没改变：*/
	public class MainActivity extends AppCompatActivity {

    private final Intent mAccessibleIntent =
            new Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS);
    private Button switchPlugin;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        switchPlugin = (Button) findViewById(R.id.button_accessible);
        updateServiceStatus();
    }

    public void onButtonClicked(View view) {
        startActivity(mAccessibleIntent);
    }


    @Override
    protected void onResume() {
        super.onResume();
        updateServiceStatus();
    }

    private void updateServiceStatus() {
        boolean serviceEnabled = false;

        AccessibilityManager accessibilityManager =
                (AccessibilityManager) getSystemService(Context.ACCESSIBILITY_SERVICE);
        List<AccessibilityServiceInfo> accessibilityServices =
                accessibilityManager.getEnabledAccessibilityServiceList(AccessibilityServiceInfo.FEEDBACK_GENERIC);
        for (AccessibilityServiceInfo info : accessibilityServices) {
            if (info.getId().equals(getPackageName() + "/.QQHongbaoService")) {
                serviceEnabled = true;
                break;
            }
        }

        if (serviceEnabled) {
            switchPlugin.setText("关闭插件");
            // Prevent screen from dimming
            getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
        } else {
            switchPlugin.setText("开启插件");
            getWindow().clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
        }
    }
	}

```

这里是MainActivity中的全部代码，是不是很少的样子，主要是实现了一个按钮去开启**ACCESSIBILITY_SERVICE**。这个插件主要就是借助**AccessibilityService**这个服务来实现。所以剩下的代码就都在这个服务中了！  

**QQHongbaoService：**
<pre><code>public class QQHongbaoService extends AccessibilityService {

    private static final String WECHAT_OPEN_EN = "Open";
    private static final String WECHAT_OPENED_EN = "You've opened";
    private final static String QQ_DEFAULT_CLICK_OPEN = "点击拆开";
    private final static String QQ_HONG_BAO_PASSWORD = "口令红包";
    private final static String QQ_CLICK_TO_PASTE_PASSWORD = "点击输入口令";
    private boolean mLuckyMoneyReceived;
    private String lastFetchedHongbaoId = null;
    private long lastFetchedTime = 0;
    private static final int MAX_CACHE_TOLERANCE = 5000;
    private AccessibilityNodeInfo rootNodeInfo;
    private List<AccessibilityNodeInfo> mReceiveNode;

    @TargetApi(Build.VERSION_CODES.KITKAT)
    public void recycle(AccessibilityNodeInfo info) {
        if (info.getChildCount() == 0) {

            if (info.getText() != null && info.getText().toString().equals(QQ_CLICK_TO_PASTE_PASSWORD)) {

                info.getParent().performAction(AccessibilityNodeInfo.ACTION_CLICK);

            }

            if (info.getClassName().toString().equals("android.widget.Button") && info.getText().toString().equals("发送")) {
                info.performAction(AccessibilityNodeInfo.ACTION_CLICK);
            }

        } else {
            for (int i = 0; i < info.getChildCount(); i++) {
                if (info.getChild(i) != null) {
                    recycle(info.getChild(i));
                }
            }
        }
    }

    @TargetApi(Build.VERSION_CODES.JELLY_BEAN)
    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {

        this.rootNodeInfo = event.getSource();

        if (rootNodeInfo == null) {
            return;
        }

        mReceiveNode = null;

        checkNodeInfo();


        /* 如果已经接收到红包并且还没有戳开 */
        if (mLuckyMoneyReceived && (mReceiveNode != null)) {
            int size = mReceiveNode.size();
            if (size > 0) {

                String id = getHongbaoText(mReceiveNode.get(size - 1));

                long now = System.currentTimeMillis();

                if (this.shouldReturn(id, now - lastFetchedTime))
                    return;

                lastFetchedHongbaoId = id;
                lastFetchedTime = now;

                AccessibilityNodeInfo cellNode = mReceiveNode.get(size - 1);

                if (cellNode.getText().toString().equals("口令红包已拆开")) {
                    return;
                }

                cellNode.getParent().performAction(AccessibilityNodeInfo.ACTION_CLICK);

                if (cellNode.getText().toString().equals(QQ_HONG_BAO_PASSWORD)) {

                    AccessibilityNodeInfo rowNode = getRootInActiveWindow();
                    if (rowNode == null) {
                        Log.e(TAG, "noteInfo is　null");
                        return;
                    } else {
                        recycle(rowNode);
                    }
                }

                mLuckyMoneyReceived = false;

            }
        }

    }


    /**
     * 检查节点信息
     */
    private void checkNodeInfo() {

        if (rootNodeInfo == null) {
            return;
        }

         /* 聊天会话窗口，遍历节点匹配“点击拆开”，“口令红包”，“点击输入口令” */
        List<AccessibilityNodeInfo> nodes1 = this.findAccessibilityNodeInfosByTexts(this.rootNodeInfo, new String[]{
                QQ_DEFAULT_CLICK_OPEN, QQ_HONG_BAO_PASSWORD, QQ_CLICK_TO_PASTE_PASSWORD, "发送"});

        if (!nodes1.isEmpty()) {
            String nodeId = Integer.toHexString(System.identityHashCode(this.rootNodeInfo));
            if (!nodeId.equals(lastFetchedHongbaoId)) {
                mLuckyMoneyReceived = true;
                mReceiveNode = nodes1;
            }
            return;
        }

    }


    /**
     * 将节点对象的id和红包上的内容合并
     * 用于表示一个唯一的红包
     *
     * @param node 任意对象
     * @return 红包标识字符串
     */
    private String getHongbaoText(AccessibilityNodeInfo node) {
        /* 获取红包上的文本 */
        String content;
        try {
            AccessibilityNodeInfo i = node.getParent().getChild(0);
            content = i.getText().toString();
        } catch (NullPointerException npe) {
            return null;
        }

        return content;
    }


    /**
     * 判断是否返回,减少点击次数
     * 现在的策略是当红包文本和缓存不一致时,戳
     * 文本一致且间隔大于MAX_CACHE_TOLERANCE时,戳
     *
     * @param id       红包id
     * @param duration 红包到达与缓存的间隔
     * @return 是否应该返回
     */
    private boolean shouldReturn(String id, long duration) {
        // ID为空
        if (id == null) return true;

        // 名称和缓存不一致
        if (duration < MAX_CACHE_TOLERANCE && id.equals(lastFetchedHongbaoId)) {
            return true;
        }

        return false;
    }

    /**
     * 批量化执行AccessibilityNodeInfo.findAccessibilityNodeInfosByText(text).
     * 由于这个操作影响性能,将所有需要匹配的文字一起处理,尽早返回
     *
     * @param nodeInfo 窗口根节点
     * @param texts    需要匹配的字符串们
     * @return 匹配到的节点数组
     */
    private List<AccessibilityNodeInfo> findAccessibilityNodeInfosByTexts(AccessibilityNodeInfo nodeInfo, String[] texts) {
        for (String text : texts) {
            if (text == null) continue;

            List<AccessibilityNodeInfo> nodes = nodeInfo.findAccessibilityNodeInfosByText(text);

            if (!nodes.isEmpty()) {
                if (text.equals(WECHAT_OPEN_EN) && !nodeInfo.findAccessibilityNodeInfosByText(WECHAT_OPENED_EN).isEmpty()) {
                    continue;
                }
                return nodes;
            }
        }
        return new ArrayList<>();
    }

    @Override
    public void onInterrupt() {

    }
}
</code></pre>
**QQHongbaoService**的全部代码也在这里，代码不多。首先，在这个服务中主要是通过**findAccessibilityNodeInfosByText**这个方法去获我们需要的节点；然后，用  **performAction(AccessibilityNodeInfo.ACTION_CLICK)** 这个方法去点击红包节点，关键思路大概就是这样！  
另外如果是口令红包，我们需要先按照上面的步骤将红包戳开，然后通过**performAction(AccessibilityNodeInfo.ACTION_CLICK)**去**点击输入口令**，最后再通过点击去发送即可实现！
  
**QQHongbaoService**需要在**AndroidManifest.xml**文件中注册，
注册的<application>节点如下图：  
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1440183-9bad264ee6690000.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

总体来看，只是将微信抢红包的代码做了少量的修改，在这里要感谢各位大神对微信抢红包源码的贡献！最后也希望这篇文章能给大家有所帮助，在抢红包大战中虐死单身狗，再也不怕你20年的单身手速了！！！  

#### 下面是源码地址：  
点击：[<font color=#0000FF>下载地址</font>](https://github.com/tiandawu/QQHongbao)
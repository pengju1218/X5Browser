“X5内核功能演示”使用说明
前言
腾讯X5内核组提供webview的部分用法用例（提供源代码）。
本文适用于初次适用TBS SDK加载X5内核的开发者对于X5内核部分api的调用情况展示。
如果你在使用X5视频播放时出现问题，请参考本文的视频播放部分。

首次编译
注意更换签名应用的my.keystore，否则会导致鉴权失败。
  
打开app后，各个webview的演示功能呈现为九宫格：
如图所示
 


一、文件选择器
当点击web页面中<input type=”file”>标签元素后，会触发WebChromeClient中的openFileChooser方法，覆盖这个方法即可以实现在android框架下的文件选择功能弹窗。在实例app中的openFileChooser位于X5WebView.java类中。
注意：为了得到ChooserActivity返回的数据，我们需要在调用FileChooser的activity中覆盖onActivityResult方法来获取FileChooser返回的结果。（详细代码见FileChooserActivity.java）
 
二、全屏播放
由于X5对视频的全屏播放做了调整，所以调用全屏的方式需要按照以下要求去做：
Bundle data = new Bundle();
data.putBoolean("standardFullScreen", true);//true表示标准全屏，false表示X5全屏；不设置默认false，
然后使用webview.putExtra(data)的方法传入开关。这样当点击视频的全屏播放时，WebChromeClient中的onShowCustomView就会被调起，这样你可以重新利用视频View布局你自己的全屏模式。（具体代码见com.example.test_webview_demo.utils.X5WebView.java与FullscreenActivity.java）
      


三、cookie的使用
（暂未开放的示例）
四、Java js的相互调用
Android webkit提供了javascript与webview框架通信的方法。一般来讲使用了webview的addJavaScriptInterface方法，从而向web页面注入一个新的Object，其中的js语法可以调用android中预先设置好的方法。（详细代码见JavaToJsActivity.java）
         
例如android端调用：
webView.addJavascriptInterface(new WebViewJavaScriptFunction() {
			
			@Override
			public void onJsFunctionCalled(String tag) {
				// TODO Auto-generated method stub
				
			}
			///////////////////////////////////////////////
			//javascript to java methods
			@JavaScriptInterface
			public void onSubmit(String s){
				Log.i("jsToAndroid","onSubmit happend!");
				JavaToJsActivity.this.msg=s;
				Message.obtain(handler, MSG).sendToTarget();
		}
}

JavaScript端的调用：
	function click_normal(){
    		Android.onCustomButtonClicked();
   		location.reload(false);
	}

五、浏览器demo
此功能向app开发者提供了一套开发自身浏览器的代码，主要展示了X5内核作为浏览器核心部件的能力。（代码详见BrowserActivity.java）


六、TBS视频裸播
TBS视频裸播是X5内核SDK提供的一个独特的功能，它在SDK层包装了一个用于播放视频的Activity，用户只要load纯视频资源的url就可以提供播放功能。（详细代码见MainActivity.java的invokeTbsVideoPlayer方法）
注意：使用时必须严格按照以下流程，否则将会导致播放失败。
1.  第一步
AndroidManifest需要如下的注册：（无需新建VideoActivity类，只需要在androidManifest中声明即可）
<activity
    android:name="com.tencent.smtt.sdk.VideoActivity"
    android:configChanges="orientation|screenSize|keyboardHidden"
    android:exported="false"
    android:launchMode="singleTask"
    android:alwaysRetainTaskState="true">
    <intent-filter>
        <action android:name="com.tencent.smtt.tbs.video.PLAY" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
说明：VideoActivity是TBS自带的组件，需要App如上配置

2.  第二步
播放视频的调用接口
通过TbsVideo的静态方法，如下：
public static boolean canUseTbsPlayer(Context context)
//判断当前Tbs播放器是否已经可以使用。
public static void openVideo(Context context, String videoUrl)
//直接调用播放接口，传入视频流的url
public static void openVideo(Context context, String videoUrl, Bundle extraData)
//extraData对象是根据定制需要传入约定的信息，没有需要可以传如null
（无需预先加载X5webview，canUseTbsPlayer中包含了X5的初始化）

 
七、网页图片
鉴于第三方同学多次提出如何获取网页图片的资源定位的问题，我们制作了获取网页图片资源的演示例。这个过程主要使用Webview的内部类HitTestResult实现。（代码见ImageResultActivity.java）
原理：当Webview发生触摸事件时，web上被激活的标签信息会被保存在HitTestResult中，其中标签与HitTestResult的一个整数保持一一对应的关系。例如：<img>-------IMAGE_TYPE
<input>------EDIT_TEXT_TYPE 等。
 
注意：ImageResultActivity.java提供了后台下载图片的方法封装SaveNetworkResource.java类，第三方app开发者可以参考。

八、TBS数据库
（暂未开放的示例）
九、Web窗口转移
Web窗口转移主要针对当前webview中使用<a>标签或者js使用openNewWindow后调起webview的一个新窗口。当新窗口建立时，可以将新建url渲染出的webview content转移到新建的webview中，实现web的窗口转移。（代码见WebViewTransportActivity.java 和 X5WebView.java）
原理：当前web要求开启新窗口时会调起WebChromeClient中的onCreateWindow方法，可以在此新建webview然后将原来的webview内容转移到新建的webview窗口。
         

十、系统内核测试
特别提供的一个用system webview构建的浏览器demo，配置方式与X5webview对齐，主要用来对比X5webview与原生webview之间的不同。（代码详见SystemWebActivity.java）
 

十一、TBS flash播放
TBS X5内核保留了对Flash播放的支持。
使用之前请确保当前的手机安装了Flash插件。
        
十二、X5内核使用时前端问题
（暂未开放的示例）

十三、安全Js Android相互调用
（暂未开放的示例）

十四、定制长按弹出功能
鉴于部分开发者希望使用X5内核的部分长按功能，而其他功能则希望自己定制，因此提供实现此需求的方法。（代码详见LongPressListenerWrapper.java 和 MyLongPressActivity.java）
需要注意：
1）当事件处理方法返回true时，X5内核的定制功能就不会启动；
2）使用HitTestResult来判定当前被点击获取焦点的H5元素类型。

          
十五、OverScroll
鉴于直接调用X5webview的getScrolly将无法准确获取y==0的情形。如果你想要X5内核准确获取webview滑到尽头的时机，请按照以下方法编写代码。（代码见类：RefreshActivity.java X5WebViewEventHandler.java）
         
开发步骤：
a)	开发者实现：在App中设置WebViewClientExtension

		// 各种设置
		if (mWebView.getX5WebViewExtension() != null) {
			// Log.e("robins", "CoreVersion_FromSDK::" +
			// mWebView.getX5WebViewExtension().getQQBrowserVersion());
			mWebView.getX5WebViewExtension().setWebViewClientExtension(
					new ProxyWebViewClientExtension() {
						@Override
						public Object onMiscCallBack(String method,
								Bundle bundle) {
							return null;
						}
						
						@Override
						public boolean onTouchEvent(MotionEvent event, View view) {
							return mCallbackClient.onTouchEvent(event, view);
						}
						
						   // 1
						public boolean onInterceptTouchEvent(MotionEvent ev, View view) {
							return mCallbackClient.onInterceptTouchEvent(ev, view);
						}	

						// 3
						public boolean dispatchTouchEvent(MotionEvent ev, View view) {
							return mCallbackClient.dispatchTouchEvent(ev, view);
						}
						// 4
						public boolean overScrollBy(int deltaX, int deltaY, int scrollX, int scrollY,
								                    int scrollRangeX, int scrollRangeY, 
								                    int maxOverScrollX, int maxOverScrollY,
								                    boolean isTouchEvent, View view) {
							return mCallbackClient.overScrollBy(deltaX, deltaY, scrollX, scrollY, 
									scrollRangeX, scrollRangeY, maxOverScrollX, maxOverScrollY, isTouchEvent, view);
						}
						// 5
					    public void onScrollChanged(int l, int t, int oldl, int oldt, View view) {
					    	mCallbackClient.onScrollChanged(l, t, oldl, oldt, view);
						}
					    // 6
					    public void onOverScrolled(int scrollX, int scrollY, boolean clampedX, 
								boolean clampedY, View view) {
					    	mCallbackClient.onOverScrolled(scrollX, scrollY, clampedX, clampedY, view);
						}
						// 7
						public void computeScroll(View view) {
							mCallbackClient.computeScroll(view);
						}
					});
b)	开发者实现：App-WebView的对应实现

// TBS: Do not use @Override to avoid false calls
	public boolean tbs_dispatchTouchEvent(MotionEvent ev, View view) {
		boolean r = super.super_dispatchTouchEvent(ev);

		return r;
	}

	// TBS: Do not use @Override to avoid false calls
	public boolean tbs_onInterceptTouchEvent(MotionEvent ev, View view) {
		boolean r = super.super_onInterceptTouchEvent(ev);

		return r;
	}
public boolean tbs_onInterceptTouchEvent(MotionEvent ev, View view) {
				boolean r = super.super_onInterceptTouchEvent(ev);
				return r;
		}
		
		protected void tbs_onScrollChanged(int l, int t, int oldl, int oldt,
				View view) {
			// TODO Auto-generated method stub
			super_onScrollChanged(l, t, oldl, oldt);
		}
		protected void tbs_onOverScrolled(int scrollX, int scrollY,
				boolean clampedX, boolean clampedY, View view) {
			// TODO Auto-generated method stub
			super_onOverScrolled(scrollX, scrollY, clampedX, clampedY);
		}
		protected void tbs_computeScroll(View view) {
			// TODO Auto-generated method stub
			super_computeScroll();	
		}
		protected boolean tbs_overScrollBy(int deltaX, int deltaY, int scrollX,
				int scrollY, int scrollRangeX, int scrollRangeY,
				int maxOverScrollX, int maxOverScrollY, boolean isTouchEvent,
				View view) {
			// TODO Auto-generated method stub
			return super_overScrollBy(deltaX, deltaY, scrollX, scrollY, scrollRangeX, scrollRangeY, maxOverScrollX, maxOverScrollY, isTouchEvent);
		}









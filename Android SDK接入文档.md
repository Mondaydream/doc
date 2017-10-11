##YiConnect Android IM_SDK接入文档 v1.0
###下载SDK 然后将如下步骤将SDK导入到项目中
	下载解压缩后得到如下文件
	|__assets
		|
		|__xiao ----->表情资源文件
		|__emoji ---->表情配置文件
	|__libs
		|
		|__YIConnect-SDK-1.0.0.jar ------>jar包
	|__example ---> Demo
	|__md ---> SDK接入文档.md

##初始化
###首先获取token和appId,然后新建一个Java Class名字叫做 MyLeanCloudApp,让它继承自 Application 类，实例代码如下:
	public class MyLeanCloudApp extends Application {

    		@Override
    		public void onCreate() {
        		super.onCreate();

        		// 初始化参数依次为 this, token, appId
        		YIConfig.initialize(this,"{{token}}","{{appId}}");
				//开发调试阶段设置为true
				YIConfig.setDebugMode(true);
				//打开通知
				YIConfig.setNotificationMode(true);
				//设置点击通知栏跳转到对应的界面
       			YIConfig.setNotification(ChatActivity.class);
    		}
	}

###初始化数据本地持久层
	//storageId不可为空,SDK根据不同的storageId为每个用户创建唯一的本地存储
	//开发者可在用户初次登录或切换用户后调用将对应的userId或其他用户的唯一标识作为storageId传入
	YIConfig.initMessageStorage(storageId);

###然后打开 AndroidManifest.xml 文件来配置 SDK 所需要的手机的访问权限以及声明刚才我们创建的 MyLeanCloudApp 类：
		<!--（必须加入以下声明）START -->
		<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
		<uses-permission android:name="android.permission.INTERNET"/>
		<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
		<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
		<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
		
		<application
		  ...
		  android:name=".MyLeanCloudApp" >
		
		  <!-- 实时通信模块 -->
		  <service android:name="com.wezhuiyi.yiconnect.im.manager.YIPushService"/>

		  <receiver android:name="com.wezhuiyi.yiconnect.im.manager.YIBroadcastReceiver">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
                <action android:name="android.intent.action.USER_PRESENT"/>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
                <action android:name="YiConnect.com.PUSH_ACTION"/>
            </intent-filter>
          </receiver>

		  <!-- 监听网络状态的广播必须要 -->
        <receiver android:name="com.wezhuiyi.yiconnect.im.manager.YIConnectNetWorkStatusReceiver">
            <intent-filter>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
                <action android:name="android.net.wifi.WIFI_STATE_CHANGED" />
                <action android:name="android.net.wifi.STATE_CHANGE" />
            </intent-filter>
        </receiver>
		</application>
##聊天发送消息
			//发送文本消息
			YIImTextMessage msg = new YIImTextMessage();
            msg.setWord(message);
            YIMessage.getInstance().createMessage(msg).send(new YIMessageStatusListener() {
                @Override
                public void onMessageStatusReceived(YIImMessage message, boolean isSuccess) {
					//判断回调对应的消息实体类是文本消息还是图片或文件...
                    if (message instanceof YIImTextMessage) {
						//word为对应的哪一条消息体
                        String word = ((YIImTextMessage) message).getWord();
						//messageId消息的唯一标志
                        String messageId = ((YIImTextMessage) message).getMessageId();
						//isSuccess为true表示消息发送成功,false表示消息发送失败(发送超时)
                    }
                }
            });

##消息重发
	//message为消息实体
	//文本消息对应YIImTextMessage,图片消息对应YIImImageMessage,文件消息对应YIImFileMessage
	YIMessage.getInstance().reSendTxtMessage(message);
##在聊天对应的Activity创建时onCreate()方法中添加接收消息监听
	YIClient.getInstance().addMessageListener(mYIMessageListener);
	
		
		//接收消息监听
		YIMessageListener mYIMessageListener = new YIMessageListener() {
        @Override
        public void onMessageTxtReceived(YINewsBean newsBean) {
            	//private String identity; ---->service表示人工客服发来的消息,yibot表示机器人发来的消息
    			//private String news;	   ---->news为消息内容(文本消息,图片或文件下载链接)
    			//private String type;	   ---->word表示文本消息,iamge表示图片消息,file表示文件消息
    			//private String reSend;
    			//private String messageDate; ---->消息对应的时间戳(服务端生成的时间)
    			//private String messageId;   ---->每一条消息对应的唯一标识
        }

        @Override
        public void onMessageImageReceived(YINewsBean newsBean) {

        }

        @Override
        public void onMessageFileReceived(YINewsBean newsBean) {

        }
    };
##在onDestroy()方法中移除接收消息监听
	//移除接收消息监听
	YIClient.getInstance().removeMessageListener(mYIMessageListener);

##转人工客服
	YIClient.getInstance().connectService();
##添加转人工状态监听
	YIClient.getInstance().addConnectSerivceStatusLinsener(YIConnectServiceStatusListener listener);
	
	/**
     * 转人工成功回调接口
     */
    void onConnectSuccessReceived();

    /**
     * 与人工客服会话结束
     */
    void onConnectClosed();
		//可调用requestGetForEvaluationSize()接口同步可提交评价项来展示对应的UI布局
		YIRequestion.requestGetForEvaluationSize();
		//提交对客服人员的服务评价
		//valuation 评价的满意值 3项 31(不满意) 32(一般) 33(满意) 5项 51(非常不满意) 52(不满意) 53(一般) 54(满意) 55(非常满意)
		YIRequestion.requestPostForEvaluations(valuation);

    /**
     * 转人工排队中接口
     *
     * @param queueNumber
     */
    void onQueueNumberReceived(String queueNumber);//排队人数---0表示只有你一个人在排队

    /**
     * 取消排队
     */
    void onCancelQueueReceived();
		//当调用了取消排队时回调

    /**
     * 转人工异常--->排队上限/已在排队....
     *
     * @param code 错误码
     * @param des  错误描述
     */
    void onConnectServiceError(String code, String des);
		//code 为 "-61008"表示当前无人工客服在线
			当无人工客服在线时可调用
			YIRequestion.requestPostForLeaveMessage(email, phone, content);
			提交留言给客服人员
		//当code 为 "-61007"表示排队人数达到上限 可通过Toast显示对应的des错误描述
		//当code 为 "-61008" 可通过Toast显示对应的des错误描述
		//当code 为 "61001"表示正在排队中... 可通过Toast显示对应的des错误描述
		//当code 为 "2"表示当前已是人工客服 可通过Toast显示对应的des错误描述y
##当转人工在排队状态时发起取消排队
	YIClient.getInstance().cancelQueueConnectService();

##添加网络状态监听
	YIClient.getInstance().addNetWorkStatusListener(mYINetWorkStatusListener);

	YINetWorkStatusListener mYINetWorkStatusListener = new YINetWorkStatusListener() {
        @Override
        public void onNetWorkConnected() {
            
        }

        @Override
        public void onNetWorkDisConnected(int code) {
            //code错误码对应的状态
				code = 24 network disconnect
				code = -1 connection refused
				code = 3000 connection unhealthy
				code = 1006 remote NotAvailable
        }
    };

##配置消息通知
###在聊天对应的界面中在onCreate()方法中添加:
	YIConfig.setNotificationMode(false);//不发消息通知
###在聊天对应的界面中在onDestroy()方法中添加:
	YIConfig.setNotificationMode(true);//设置发送消息通知

##本地消息操作
###查询所有消息
	YIMessage.getInstance().getAllMessage();
	返回值为 ArrayList<YINewsBean>
###分页查询消息
	/**
	 *pageCount表示查询一页查询到的消息条数(ps: 当pageCount为10 一页查询10条消息)
	 *pageSize 表示对应的页数(当pageSize为1 pageCount为10 查询到第一页对应的10条消息)
	 *注意:进行分页查询多次调用时pageCount的值要以第一次调用传入的值为准不能多次传入不同的值
	 *	   pageSize的值需要递增,第一次为1第二次为2....
	 */
	YIMessage.getInstance().getMessageWithPage(pageSize, pageCount);
	返回值为 ArrayList<YINewsBean>
###查询最近一条消息
	YIMessage.getInstance().getMessageForLast();
	返回值为 ArrayList<YINewsBean>
###从服务端同步消息并查询出最近一条消息
	YIMessage.getInstance().synchronizeMessageFromRemote(YIMessage.onGetMessageForLastListener listener);
	//方法回调
	onSuccess(ArrayList<YINewsBean> newsBeen);//表现同步成功
	onFailure();//表示同步失败
###查询未读消息条数
	YIMessage.getInstance().getUnreadMessageCount()
	返回值为 int count
###更新消息状态(未读--->已读)
	//在进入聊天界面后调用进行消息状态更新
	YIMessage.getInstance().updateUnReadMessage();
###根据messageId删除对用的消息
	YIMessage.getInstance().deleteMessageWithMessageId(messageId);
	返回值为boolean类型 true表示删除成功,false表示删除失败
###删除所有的消息
	YIMessage.getInstance().deleteAllMessage();
	返回值为boolean类型 true表示删除成功,false表示删除失败
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
###首先获取token和appId,然后新建一个Java Class名字叫做 MyApplication,让它继承自 Application 类，实例代码如下:
	public class MyApplication extends Application {

    		@Override
    		public void onCreate() {
        		super.onCreate();

        		// 初始化参数依次为 this, token, appId
        		YIConfig.initialize(this,"{{token}}","{{appId}}");
				//开发调试阶段设置为true 默认为false
				YIConfig.setDebugMode(true);
				//打开通知 默认为false
				YIConfig.setNotificationMode(true);
				//设置点击通知栏跳转到对应的界面
       			YIConfig.setNotification(ChatActivity.class);
				//设置是否缓存多媒体数据 默认为false不缓存
                YIConfig.setMediaMessageCacheEnabled(true);
				//设置超时时长 默认是20000L
				YIConfig.setTimeOutDuration(10000L);
    		}
	}

###与服务端建立长连接
	//serviceId不可为空,SDK根据不同的serviceId为每个用户创建唯一持久层
	//开发者可在用户初次登录或切换用户后调用将对应的userId或其他用户的唯一标识作为storageId传入
	YIService.getInstance(serviceId).connectWithService(new YIServiceCallback() {
                    @Override
                    public void done(YIException e) {
                        if (e == null) {
                            //当e == null时表明已经与service成功建立连接,可与service正常进行通信
                        } else {
                            Log.e("YIConnect", "" + e.getMessage());
                        }
                    }
                });

###然后打开 AndroidManifest.xml 文件来配置 SDK 所需要的手机的访问权限以及声明刚才我们创建的 MyLeanCloudApp 类：
		<!--（必须加入以下声明）START -->
		<uses-permission android:name="android.permission.INTERNET"/>
    	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    	<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
    	<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
		
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
##从服务端同步聊天消息到本地消息持久层
		YIMessage.getInstance().synchronizeMessageFromRemote(new YISyncMessageFromRemoteCallback() {
            @Override
            public void done(YIException e) {
                if (e == null) {
                    //当e == null时表明同步消息成功
                } else {
                    //同步消息失败
                }
            }
        });
##通过服务器查询历史消息
		/**
     	 * 通过服务器查询历史消息
     	 * @param timestamp 查询时间戳以下的历史消息
     	 * @param limmit    消息条数 最新的纪录向前查询limit条数
     	*/
		YIRequestion.getHistoryMessageFromServiceAfterTimestamp(final String timestamp, final int limmit, final YIGetHistoryMessageCallback callback)
###通过服务器查询历史消息记录，从最新的纪录向前查询limit条数,最多查询15天的消息
		/**
     	 * 通过服务器查询历史消息记录，从最新的纪录向前查询limit条数,最多查询15天的消息
     	 * @param limmit 条数
     	 * @param callback 回调
     	*/
		YIRequestion.getHistoryMessagesFromServerWithLimmit(int limmit, YIGetHistoryMessageCallback callback)
##聊天发送消息
			//发送文本消息 (SDK限制文本最大长度为500个字符,超过将发送失败)
			YIImTextMessage msg = new YIImTextMessage();
            msg.setWord(message);
			msg.setOnMessageIdChangeListener(new YIImMessage.onMessageIdChangeListener() {
                    @Override
                    public void onChange(String messageId) {
						//开发者可监听每一条消息对应的messageId
                        Log.e("YIConnect", "===messageId===" + messageId);
                    }
                });
			
			//发送图片消息
			YIImImageMessage imageMessage = new YIImImageMessage();
                        imageMessage.setImageType("image");
                        imageMessage.setImageName("{{imageName}}");
						//imageData 可以是byte[] 也可以是filePath
                        imageMessage.setImageData("{{imageData}}");
						//或者图片的超链接
						imageMessage.setImageUrl("{{imageUrl}}");
			...

            YIMessage.getInstance().createMessage(msg).send(new YIMessageStatusCallback() {
                    @Override
                    public void done(YIImMessage imMessage, YIException e) {
						//当e == null表示消息发送成功
                        if (e == null) {
                            if (imMessage instanceof YIImTextMessage) {
                                //文本消息
                            }
                        } else {
                            if (imMessage instanceof YIImTextMessage) {
                                //文本消息
                            }
                        }
                    }
                });
	
##消息重发
	//message为消息实体
	//文本消息对应YIImTextMessage,图片消息对应YIImImageMessage,文件消息对应YIImFileMessage
	//message实体必须带有对应要重发的消息的messageId
	YIMessage.getInstance().reSendTxtMessage(message);
##在聊天对应的Activity创建时onCreate()方法中添加接收消息监听
	YIClient.getInstance().addMessageListener(mYIMessageListener);
	
		
		//接收消息监听
		YIMessageListener mYIMessageListener = new YIMessageListener() {
        @Override
        public void onMessageReceived(YINewsBean newsBean) {
            //identity;消息来源的身份标识 user service
            //messageType;
			//sendStatus;
			//messageDate;
			//messageId;
			//serviceId;
			//sessionId;
			//mYIImTextMessage;
			//mYIImImageMessage;
			//mYIImFileMessage;
        }
    };
##在onDestroy()方法中移除接收消息监听
	//移除接收消息监听
	YIClient.getInstance().removeMessageListener(mYIMessageListener);

##转人工客服
	YIClient.getInstance().connectToCustomerService(new YIConnectCustomerServiceCallback() {
                        @Override
                        public void done(final YIException e) {
                            if (e != null) {
								//当e != null时表示转人工超时或失败,开发者可再次发起转人工
                                Log.e("YIConnect", "==connectToCustomerService==" + e.getMessage());
                            } else {
								//当e == null时表示转人工成功
                            }
                        }
                    });
###转人工客服(携带开发者定义的自定义参数)
	YIClient.getInstance().connectToCustomerServiceWithConfig(config, new YIConnectCustomerServiceCallback() {
                    @Override
                    public void done(YIException e) {
						if (e != null) {
								//当e != null时表示转人工超时或失败,开发者可再次发起转人工
                                Log.e("YIConnect", "==connectToCustomerService==" + e.getMessage());
                            } else {
								//当e == null时表示转人工成功
                            }
                    }
                });
##添加转人工状态监听
	YIClient.getInstance().addConnectToCustomerServiceStatusLinsener(new YIConnectCustomerServiceStatusListener() {
            @Override
            public void onConnectSuccessReceived(YIServiceInfoBean infoBean) {
               	//转人工成功回调
			   	//YIServiceInfoBean封装了人工客服的相关信息
            }

            @Override
            public void onConnectClosed(YIServiceInfoBean infoBean) {
                //在人工客服状态时断开人工客服连接回调
            }

            @Override
            public void onCancelQueueReceived() {
                //取消排队回调
            }

            @Override
            public void onConnectServiceStatus(YIServiceInfoBean infoBean) {
                //转人工客服时的状态
				//int status = infoBean.getStatus()
				//ERROR_MSG_RETRANSMIT = -61009,消息重传
				//ERROR_NO_SERVICE = -61008,没有客服在线 
  				//ERROR_REJECT_ENQUEUE = -61007,拒绝排队 
				  ERROR_SERVICE_BUSY = -61006,坐席服务人数达到最大值 
  				  ERROR_SERVICE_OFFLINE = -61005,坐席处于下线状态 
				  ERROR_SESSION_WRONG = -61004,会话错误，不存在该会话  
				  ERROR_UNKNOWN_PACKET = -61003,无法解析的Json通信数据包  
				  ERROR_SYSTEM_WRONG = -61002,系统异常，主要指的是yiserver和kv,chatProxy之间通信异常     
				  ERROR_NOT_READY = -61001,没有收到配置，系统处于非工作状态    
				  ERROR_NO_ERROR = 0,正确操作  
				  WARN_ALREADY_ONQUEUE = 61001,用户已经在排队
				  WARN_ALREADY_INSERVICE = 61002,用户已经在和坐席沟通服务
				  WARN_NOT_ONDUEUE = 61003,用户不在排队队列
				  WARN_NOT_INSERVICE = 61004,关闭会话时，提醒用户已经不在和坐席沟通服务
				  WARN_ALREADY_ONLINE = 61005,坐席上线时，提醒坐席已经处于上线状态
				  WARN_ALREADY_OFFLINE = 61006,坐席下线时，提醒坐席已经下线
				  SERVICE_WITH_NO_QUEUE = 71001,坐席对队列为空（不需要排队）
            }
        });
##检测转人工客服排队状态
		YIClient.getInstance().checkConnectToCustomerServiceQueueStatus(new YIConnectCustomerServiceQueueStatusListener() {
                    @Override
                    public void onQueueStatus(String queueNumber, YIException e) {
					   //当e != null时表示检测转人工客服排队状态超时或失败,开发者可再次发起检测转人工客服排队状态
                       //当"0".equals(queueNumber)为true时,表示正在排队中且在你之前有一位正在与客服交流中
					   //当"1".equals(queueNumber)为true时,表示正在排队中且前面有一人正在排队
					   //以此类推....
                    }
                });
##当转人工在排队状态时发起取消排队
	YIClient.getInstance().cancelQueueConnectToCustomerService(new YICancelQueueConnectCustomerServiceCallback() {
                    @Override
                    public void done(YIException e) {
                        //当e != null时表示取消排队超时或失败,开发者可再次发起转人工
                    }
                });
##结束与人工客服聊天
	YIClient.getInstance().closeSessionToCustomerService(new YICloseSessionToCustomerServiceCallback() {
                    @Override
                    public void done(YIException e) {
                        //当e != null时表示结束与人工客服聊天超时或失败,开发者可再次发起转人工
                    }
                });
##客服会话结束提交评价
		/**
     	 * 客服会话结束提交评价
     	 *
     	 * @param yiServiceInfoBean
     	*/
	YIRequestion.requestPostForEvaluations(YIServiceInfoBean yiServiceInfoBean, YISubmitForEvaluationsCallback callback)
##客服不在线提交留言
		/**
     	 * 客服不在线,提交留言
     	 *
     	 * @return 是否提交成功的状态
     	*/
		YIRequestion.requestPostForLeaveMessage(YILeaveMessageBean yiLeaveMessageBean, YISubmitForLeaveMessageCallback callback)
##添加server端发来请求监听 
	YIClient.getInstance().addServiceRequestConnectCallback(new YIServiceRequestCallback() {
            @Override
            public void done(YIServiceInfoBean infoBean) {
				
   			}
        });
##添加网络状态监听
	YIClient.getInstance().addNetWorkStatusListener(mYINetWorkStatusListener);

	YINetWorkStatusListener mYINetWorkStatusListener = new YINetWorkStatusListener() {
        @Override
        public void onNetWorkConnected(int code) {
            //code = 10001 network connecting
			//code != 10001 network connected
        }

        @Override
        public void onNetWorkDisConnected(int code) {
            //code错误码对应的状态
				code = 24 network disconnect
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
		return ArrayList<YINewsBean>
###查询未读消息
	YIMessage.getInstance().getUnreadMessage();
		return ArrayList<YINewsBean>
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
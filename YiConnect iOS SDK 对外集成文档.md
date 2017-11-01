# YiConnect iOS SDK 对外集成文档
##使用提示
本文是 YiConnect iOS SDK 的标准集成指南文档。 匹配的 SDK 版本为:V1.0.0及以后版本。

##产品功能说明

YiConnect SDK 可以让你的应用迅速支持追一的客服机器人以及人工客服通信服务。

##集成步骤
* 将Framework复制到工程中，即可开始使用SDK。
* 增加相关的 Framework 依赖
  
  * lisqlite3
  * CFNetwork
  * Foundation
  * Security
  * libicucore
  
* 在 AppDelegate.m 引用头文件的位置
 
```
// 引入 YiConnectSDK 功能所需头文件
#import <YiConnectSDK/YiService.h>
```


## SDK 主要接口说明
YiConnectLaunchConfig 类：启动配置模型类。

YiCustomerService 类：从机器人转接人工的回调模型类。

YiSetup 类：SDK 启动类，包含启动的所有接口。

YiService 类：核心通信类，包含核心通信的所有接口。

YiMessage 类：通信内容类，包含通信内容的所有字段。


### method - setupWithConfig

#### 接口定义：
+(void)setupWithConfig:(nonnull YiConnectLaunchConfig *)config;<br>

#### 接口说明：

初始化接口。建议在 application:didFinishLaunchingWithOptions: 中调用。

#### 参数说明：

* config：YiConnectLaunchConfig 类的实例。

#### 调用示例：

```

 YiConnectLaunchConfig *config = [[YiConnectLaunchConfig alloc] init];
    config.YiAppID = @"Your YiappID";
    config.YiToken = @"Your YiToken";
    [YiSetup setupWithConfig:config];

```


### method - serviceWithSeriveId

#### 接口定义：
+(instancetype _Nullable )serviceWithSeriveId:(nonnull NSString *)serviceId;<br>

#### 接口说明：

初始化核心通信实例的接口，建议入客服聊天页面之前创建好通信实例。

#### 参数说明：

* serviceId：实例的唯一标志符 不可为空!

#### 调用示例：

```
//通信实例的标志符要保持唯一性，推荐每一个用户对应一个相应通信标志符。
 self.yiService = [YiService serviceWithSeriveId:self.userTextView.text];

```

### method - connectWithHandler

#### 接口定义：
-(void)connectWithHandler:(nonnull YiConnectResultHandler)handler<br>

#### 接口说明：

通过通信核心实例建立起通信连接。

#### 参数说明：

* handler：建立通信之后的回调.

#### 调用示例：

```
 self.yiService = [YiService serviceWithSeriveId:self.userTextView.text];
    [self.yiService connectWithHandler:^(BOOL successed, NSError * _Nullable error) {
        if (successed) {
          nslog(@"可以开始与客服沟通");
        }
    }];

```

### method - send:

#### 接口定义：
-(void)send:(nonnull YiMessage *)message
    handler:(nullable YiConnectResultHandler)handler;<br>

#### 接口说明：

发送消息

#### 参数说明：

* message：发送的消息实例。
* handler：发送消息之后的回调。

#### 调用示例：

```
//_yiService 实例的状态尽量是已经建立起链接的情况。
 YiMessage *yimessage = [[YiMessage alloc] init];
    yimessage.content = text;
    [_yiService send:yimessage handler:^(BOOL successed, NSError * _Nullable error) {
        if (successed) {
            NSLog(@"发送成功");
        }
    }];

```

### method - connectToCustomerService:

#### 接口定义：
-(void)connectToCustomerService:(nonnull YiCustomerServiceHandler)handler<br>

#### 接口说明：

转接到人工客服接口

#### 参数说明：

* handler：启动转接到回调。

#### 调用示例：

```
//人工客服存在在线以及不在线的多种情况，转接人工成功的消息需要开发实现转接人工成功的回调协议函数
[_yiService connectToCustomerService:^(YiCustomerService * _Nullable status, NSError * _Nullable error) {
        NSLog(@"开启转人工");
    }];

```

SDK 更加具体的用法，请参考头文件的函数说明，以及 Demo。
#end


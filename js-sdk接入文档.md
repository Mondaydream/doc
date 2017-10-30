# JS SDK接入文档
<!-- vscode-markdown-toc -->
* 1. [引入代码script](#script)
* 2. [初始化方法init](#init)
* 3. [事件event](#event)
	* 3.1. [未读消息变更onNewMessageCount](#onNewMessageCount)
	* 3.2. [接收一条未读客服消息onMessage](#onMessage)
* 4. [咨询信息接入setData【开发中】](#setData)
	* 4.1. [客户信息接入customer](#customer)
	* 4.2. [商品信息接入product](#product)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

##  1. <a name='script'></a>引入代码script

    <script>
      var _yic = _yic || [];
      (function () {
          var yis = document.createElement('script');
	      // sdk资源链接[该数据由系统生成]
          yis.src = 'https://yiconnect.wezhuiyi.com/public/im-chat/static/web-sdk.js?756a47fc475b1fd22b0d33f283fa4091';
          var s = document.getElementsByTagName('script')[0];
          s.parentNode.insertBefore(yis, s);
      })();
    </script>

##  2. <a name='init'></a>初始化方法init
    _yic.push(['init', {
        el: 'kefu-btn',                 // 绑定按钮ID
        token: 'xxx',                   // 加密token[该数据由系统生成]
        src: 'webChat',                 // 聊天地址[该数据由系统生成]
        style: {
            className: 'chat-window',   // 自定义类名
            width: '330px',             // 宽度
            height: '480px',            // 高度

            // 距离（也可以设置top、left）
            right: '40px',
            bottom: '0',
        },
        themeColor:'#4ba0ff',           // 默认主题色，优先级比后台设置主题色高，格式要求为 #xxxxxx 或 rgb(x,x,x)
        noYiBot: false,                 // 禁止机器人会话（自动转人工，若无人工则显示留言窗口）
        tag: '',                        // 优先技能（客服组）
        ls: '',                         // 优先坐席
        showPop: true,                  // 是否显示新消息气泡
        pop: {
            className: 'chat-pop',      // 自定义类名
            offset: {                   // 偏离位置(值为带单位的距离px,em,%)
                top: '-87px',
                left: '-250px',
            }
        },
    }]);

    // 调用代码示例
    _yic.push(['init', {
        el: 'kefu-btn',
        token: 'RjNPWT7chxE05nGwpE4JtQQfDWAaSBCr6HO3RTb8fmU',   
        src: 'webChat',            
    }]);

##  3. <a name='event'></a>事件event
###  3.1. <a name='onNewMessageCount'></a>未读消息变更onNewMessageCount
    _yic.push(['onNewMessageCount', function(num){
        document.getElementById('msg-count').innerHTML = num;
    }]);

###  3.2. <a name='onMessage'></a>接收一条未读客服消息onMessage
    _yic.push(['onMessage', function(data){
        console.log(data);
    }]);

    data: {
        seq,              // 消息唯一标识
        word,             // 消息内容
        timestamp,        // 时间戳
        serviceName,      // 客服名称
        serviceID,        // 客服ID
        serviceAvatar,    // 客服头像
        userID,           // 接收用户ID
    }

##  4. <a name='setData'></a>咨询信息接入setData
说明：
1. 所有外链资源都必须是已配置可信任的域名来源
2. 链接引入的聊天端可在url上设置大部分字段
###  4.1. <a name='customer'></a>客户信息接入customer

<!--| 字段名 | 类型 | 值 | 必填 | 说明 | 是否允许链接
|-------|------|----|------|-----|
| cName | String | 名称 | 否 |  |
| 字段名 | 类型 | 值 | 必填 | 说明 |
| 字段名 | 类型 | 值 | 必填 | 说明 |-->

    _yic.push(['setData', {
        customer: {
            cID: String,        // 外围系统用户ID
            cName: String,      // 名称
            cEmail: String,     // 邮箱
            cPhone: String,     // 手机
            cRemark: String,    // 备注
            cLink: URL,         // 客户详情（资源外链，来源配置）
            extends: Object,    // 自定义字段
        }
    }]);

    // 自定义字段示例(key,value只能是字符串类型)
    extends: {
        love: '鹿晗',
        '年龄': '22',
    }

    // URL引入示例(customer具体参数需要encode)
    xxx?token=xxx&customer={"cID":"1","cName":"2"}&sign=xxx

###  4.2. <a name='product'></a>商品信息接入product
    _yic.push(['setData', {
        product: {
            title: String,      // 标题
            desc: String,       // 描述
            src: URL,           // 链接（来源配置）
            image: URL,         // 图片链接（来源配置）
            extends: Object,    // 自定义字段            
        }
    }]);

    // 自定义字段示例(key,value只能是字符串类型)
    extends: {
        '来源': '首页活动',
        '价格': '￥19.99',
    }

    // URL引入示例(product具体参数需要encode)
    xxx?token=xxx&product={"title":"1","desc":"2"}&sign=xxx

<!--### 外围系统接入resource
    _yic.push(['setData', {
        resource: {
            ...自定义参数
        }
    }]);

    需要在后台配置资源请求url和token-->
# JS SDK接入文档
<!-- vscode-markdown-toc -->
* 1. [引入代码script](#script)
* 2. [初始化方法init](#init)
* 3. [事件event](#event)
	* 3.1. [未读消息变更onNewMessageCount](#onNewMessageCount)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

##  1. <a name='script'></a>引入代码script

    <script>
      var _yic = _yic || [];
      (function () {
          var yis = document.createElement("script");
	      // sdk资源链接[该数据由系统生成]
          yis.src = "https://yiconnect.wezhuiyi.com/public/im-chat/static/web-sdk.js?756a47fc475b1fd22b0d33f283fa4091";
          var s = document.getElementsByTagName("script")[0];
          s.parentNode.insertBefore(yis, s);
      })();
    </script>

##  2. <a name='init'></a>初始化方法init
    _yic.push(['init', {
        el: 'kefu-btn',// 绑定按钮ID
        token: 'RjNPWT7chxE05nGwpE4JtQQfDWAaSBCr6HO3RTb8fmU',// 加密token[该数据由系统生成]
        src: 'https://yiconnect.wezhuiyi.com/webChat',// 聊天地址[该数据由系统生成]
        style: {
            className: 'chat-window',// 自定义类名
            width: '330px',// 宽度
            height: '480px',// 高度
            // 距离（也可以设置top、left）
            right: '40px',
            bottom: '0',
        },
        noYiBot: false,// 禁止机器人会话（自动转人工，若无人工则显示留言窗口）
        tag: '',// 优先技能（客服组）
        ls: '',// 优先坐席
    }]);

##  3. <a name='event'></a>事件event
###  3.1. <a name='onNewMessageCount'></a>未读消息变更onNewMessageCount
    _yic.push(['onNewMessageCount', function(num){
        document.getElementById('msg-count').innerHTML = num;
    }]);

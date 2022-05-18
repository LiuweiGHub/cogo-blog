{
  "date": "2022.05.18  09:00",
  "tags": ["消息中间件","Rabbitmq"],
  "description":"使用过Rabbitmq的同学应该都知道，Rabbitmq官方提供了一个比较好用的管理后台，当我们做一些测试或者线上紧急修复操作时，可以要求管理员手动向某个队列中发布消息。但问题来了，后台只能一条一条发布，而现在的需求是成全上万条，该如何快速处置呢？"
}



### 思路一：当然是业务自己写脚本，上线执行，修复数据

- 优点：

  - 脚本经过测试，逻辑得到验证，更加稳妥

- 缺点：

  - 流程长，耗时久，如果是紧急情况，损失较大

  

### 思路二：MQ PAAS平台提供批量发布功能

- 优点：
  - 平台化，操作便携
- 缺点：
  - 需投入开发资源
  - 日常维护较多，使用频率较低
  - 需做权限控制，容易被滥用或误用

### 思路三：浏览器Console Js编码 快速实现

- 优点 ：
  - 快速止损
- 缺点：
  - 要求js编码能力



思路一和二都比较好理解，不做阐述。我们以思路三举例说明：

例：某业务部门收到用户大量投诉，需紧急修复10000条数据，修复方式是重新发布mq消息，但此时管理员后台只支持一条一条发布，一万条手动补发 不现实。此时，作为一名比较懒的程序员，投机取巧的方法就来了。**console中写个forEach不就解决了吗？**

```js
// 业务提供需修复数据关键信息
var arr = [518884581, 4169251270,2561025485,520451528,1039552993,4940998602,39037,2577677322,4944405564]

// 拼装消息体+header关键信息，循环Ajax请求搞定
// 可以先手动发一条，获取header和payload结构
arr.forEach(function (spaceId) {
    var settings = {
        "url": "http://mqm5.haodf.net/api/exchanges/teamcasecenter/amq.default/publish",
        "method": "POST",
        "timeout": 0,
        "headers": {
            "Content-Type": "application/json"
        },
        "data": JSON.stringify({
            "vhost": "teamcasecenter",
            "name": "amq.default",
            "properties": {
                "delivery_mode": 1,
                "headers": {}
            },
            "routing_key": "teamcasecenter_DelaySkuConsumer_teamcasecenter",
            "delivery_mode": "1",
            "payload": '{"data":{"spaceId":'+spaceId+'}, "toc":0}',
            "headers": {},
            "props": {},
            "payload_encoding": "string"
        }),
    };
    $.ajax(settings).done(function (response) {
        console.log(response);
    });
})

// 当然还需要授权，此时输入管理员账号密码，等待执行结果即可

// 发布完成，业务去确认数据是否已经全部修复
```



### 总结

思路三，虽然非常规手段，这里并不推荐。如果业务这种需求很多，那么提供个PAAS平台功能才是主流。这里只是记录这种解决问题的思路（骚操作）而已~~~


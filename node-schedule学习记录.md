# node-schedule学习记录
最近在做的一个项目中，有一个需求是定期执行一个服务，简单的Google了一下发现node.js有一个模块叫做node-schedule，功能已经可以覆盖大部分应用场景，遂学习并记录一下使用过程。  

[node-schedule](https://www.npmjs.com/package/node-schedule)

## 概述

Node-Schedule的调度方式是基于时刻，而不是基于时间间隔的，这代表着，虽然你能很轻松的使用schedule模块实现类似于"每隔5分钟执行一次某任务"这种需求，但是相比较而言，setInterval可能更适合这种需求。但是如果你想实现类似"每个月第三个周的星期二的晚八点四十分和九点执行这个程序"，那么，使用Node-Schedu就哦了。

## 使用cron进行任务安排

虽然提供了非常全面的rule设定方式，但是Node-Schedule也支持使用更加直观且简洁的cron表达式来指定任务计划。  
cron是Unix下的一个工具，可以使用crontab进行配置。cron表达式是一个字符串，字符串以5或6个空格隔开，分为6或7个域，每一个域代表一个含义，其含义如下描述：
```
*  *  *  *  *  *
┬  ┬  ┬  ┬  ┬  ┬
│  │  │  │  │  │
│  │  │  │  │  └─ day of week (0 - 7) (0 or 7 is Sun)
│  │  │  │  └──── month (1 - 12)
│  │  │  └─────── day of month (1 - 31)
│  │  └────────── hour (0 - 23)
│  └───────────── minute (0 - 59)
└──────────────── second (0 - 59, OPTIONAL)
```
|字段|允许值|允许的特殊字符|
|---|---|---|
|秒（Seconds）|0~59的整数|, - * / 四个字符|
|分（Minutes）|0~59的整数|, - * / 四个字符|
|小时（Hours）|0~23的整数|, - * / 四个字符|
|日期（DayofMonth）|1~31的整数（但是你需要考虑你月的天数）|,- * ? / L W C 八个字符|
|月份（Month）|1~12的整数或者 JAN-DEC|, - * / 四个字符|
|星期（DayofWeek）|1~7的整数或者 SUN-SAT|（1=SUN）|, - * ? / L C # 八个字符|
|年(可选，留空)（Year）|1970~2099|, - * / 四个字符|

特殊字符的含义如下：
|字符|含义|
|---|---|
|*|表示匹配任意值，若分钟字段为*，则表示每分钟都会触发|
|?|只能应用于DoM和DoW两个域，用来指明‘没有特定的值’|
|-|表示范围，例如分钟域的`5-20`表示5分到20分每分钟都触发|
|/|例如分钟域的`0/5`，表示从0分钟开始触发，每5分钟触发一次|
|,|用来连接枚举值，例如分钟域的`5,20`表示5分和20分时触发一次|
|L|表示最后，只能用于DoM和DoW两个域，表示最后一周/一月|
|W|表示有效工作日，同上|
|#|用于确定星期，只能出现在week字段，表示第几个星期几|

### cron与Node-Schedule

Node-Schedule不支持cron表达式中`L W #`三种字符的用法，其他的常用功能则支持的很好
一些样例：
```
每分钟的第30秒触发：        '30 * * * * *'
每小时的1分30秒触发 ：      '30 1 * * * *'
每天的凌晨1点1分30秒触发 ： '30 1 1 * * *'
每月的1日1点1分30秒触发 ：  '30 1 1 1 * *'
2018年的1月1日1点1分30秒触发 ：'30 1 1 1 2018 *'
每周1的1点1分30秒触发 ：    '30 1 1 * * 1' 
```

## Job对象

在Node-Schedule中，每一个调度的任务都由一个Job对象来表示。我们可以手动创建一个任务，然后使用schedule()方法绑定一个计划，或者使用更简单的方法scheduleJob()来绑定计划。

Job对象继承自EventEmitter对象，每次运行时会发送一个事件；他们还会在每次调度运行时发出调度(`scheduled event`)事件，在取消时发出取消(`cancaled event`)事件，两个事件都接收JavaScript的Date对象作为参数。需要注意的是，如果使用scheduleJob()方法来快速绑定任务，你会错过第一个schedule事件。

## 一些案例

### 基于cron表达式的调度
```js
//使用cron表达式安排任务
let cron = '40-50 * * * * *'
console.log('任务开始时间：',new Date());
let j = schedule.scheduleJob(cron,function(){
    console.log(`现在时间${Date()}`);
})
```
输出结果：   
![cron](/img/cron.png)
### 基于Date的调度
**注意，在JS中，0代表1月，11代表12月**
```js
let date = new Date(2018, 6, 10, 16, 50, 10);
let j = schedule.scheduleJob(date, function(){
  console.log(`现在时间${Date()}`);
});
```
输出结果:  
![date](/img/date.png)

### Recurrence-Rule调度模式
```js
let rule = new schedule.RecurrenceRule();
// rule.dayOfWeek = 2;
// rule.month = 3;
// rule.dayOfMonth = 1;
// rule.hour = 1;
// rule.minute = 42;
rule.second = 30;
console.log('任务开始时间：',new Date());
let j = schedule.scheduleJob(rule, function(){
    console.log(`触发时间:${Date()}`);
});
```
![RecurrenceRule](/img/RecurrenceRule.png)

### 对象文法调度模式
```js
let j = schedule.scheduleJob({ hour: 17, minute: 1 }, function () {
    console.log(`触发时间:${Date()}`);
});
```

### 区间定时模式
```js
let startTime = new Date(Date.now() + 5000);
let endTime = new Date(startTime.getTime() + 5000);
console.log(`任务开始时间:${Date()}`);
let j = schedule.scheduleJob({ start: startTime, end: endTime, rule: '*/1 * * * * *' }, function(){
    console.log(`触发时间:${Date()}`);
});
```
输出结果:  
![interval](/img/interval.png)

## 取消作业
- `job.cancel(reshedule)`  
    - 该方法可以取消任何作业  
    - `j.cancel();`  
        - 所有的调度都会被取消。如果传入的reschedule参数为true，那么调度会重启  

- `job.cancelNext(reshedule)`  
    - 此方法使下一个计划的调用或作业无效。将参数reschedule设置为true时，之后会重新安排Job。  

- `job.reschedule(spec)`  
    - 此方法取消所有挂起的调用，并使用给定的规范再次将Job重新注册为new。成功/失败时返回true / false。  

- `job.nextInvocation()`  
    - 此方法返回此Job的计划下一个触发时刻对应的Date对象。 如果未计划调用，则该方法返回null。  

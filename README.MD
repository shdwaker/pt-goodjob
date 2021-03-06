
部署
---------------------
#### 1. 主要结构说明

```
1. goodjob-schedule : 调度机，部署2个节点以上
2. goodjob-web : 管理控台，部署一个节点
3. goodjob-example : 接入端事例
4. goodjob-node : 执行机（接入端）引用jar
5. config : sql、缺失的jar、图片等

```

#### 2. 依赖

```
1. mysql  --- config目录下有sql
2. zookeeper --- 网上很多搭建教程

```

#### 3. 本地运行

```
1. 启动example的ExampleApplication.main
2. 启动schedule的DubboWithSchdSbApplication.main
3. 启动web的Main.main

```

#### 4. 发布

```
config/bin目录有启动和停用脚本
把start.sh 和 stop.sh跟打好的jar放到一个目录即可
如：把goodjob-schedule-1.0.2-SNAPSHOT.jar、start.sh、stop.sh放到同一个目录
启动: sh start.sh
停止: sh stop.sh
```


Quick Started
---------------------
#### 1. 依赖

```
<dependency>
    <groupId>com.wacai.pt</groupId>
    <artifactId>goodjob-node</artifactId>
    <version>1.0.2-SNAPSHOT</version>
</dependency>

```

#### 2. spring 配置

```
<dubbo:application name="${application.name}"  />
<dubbo:provider loadbalance="random" />
<dubbo:registry protocol="zookeeper" address="${dubbo.zk.servers}" client="zkclient" group="${dubbo.zk.group}"/>
<dubbo:protocol name="dubbo" />
<!--   ==============上诉为dubbo通用配置，如有则不用配==================================    -->


<bean id="failHandle" class="com.wacai.pt.goodjob.node.file.FailHandle"/> <!-- 处理服务连接不上等情况 -->
<dubbo:reference id="jobExecuteRespService" timeout="40000" interface="com.wacai.pt.goodjob.remote.service.JobExecuteRespService"/>
<!--  ============== 引用调度回调服务，必须项，超时设置为40秒 ===========================    -->

<!--  ************** 以上为通用配置 *************************************    -->




<!--  ************** 以下为业务实现 *************************************    -->

//统计日报任务 -- 配置任务1
<bean id="statisticsDayJob" class="com.wacai.schedule.monitor.StatisticsDayJob"/> <!-- 这是你唯一需要的实现 -->
<dubbo:service group="coeus_dailies" interface="com.wacai.pt.goodjob.remote.service.JobExecuteService" ref="statisticsDayJob" timeout="10000" retries="0" />

//统计周报任务 -- 配置任务2
<bean id="statisticsWeekJob" class="com.wacai.schedule.monitor.StatisticsWeekJob"/> <!-- 这是你唯一需要的实现 -->
<dubbo:service group="coeus_weekly" interface="com.wacai.pt.goodjob.remote.service.JobExecuteService" ref="statisticsWeekJob" timeout="10000" retries="0" />
<!--  ============== 服务实现，请务必按照此配置. group命名规则为【项目code + 业务】，用于区分调度任务 =======    -->

...
```


#### 3. 代码实现

```
package com.wacai.schedule.monitor;

import com.wacai.pt.goodjob.node.service.impl.AbstractJobExecuteService;
import com.wacai.pt.goodjob.node.bean.ExecuteContext;

public class StatisticsDayJob extends AbstractJobExecuteService {

    private static Logger logger = LoggerFactory.getLogger(StatisticsDayJob.class);
    @Autowired
    private StatisticsReportService statisticsReportService;

    /**
    * 如果是分片任务请从executeContext 获取param参数
    * 一般原则param为json
    * @param executeContext
    * @throws Throwable
    */
    @Override
    public void doExecute(ExecuteContext executeContext) throws Throwable {
        //这里触发你的job, 如果有异常抛出，会被认为job执行失败
        //executeContext.execMsg 可以存放业务执行数据，限制4000长度，会在控台显示，如果此方法抛异常则显显示异常信息

        statisticsReportService.sendSubscriptionMail(); //你要触发的方法
    }
}
```


设计
---------------------
<https://github.com/wacai/pt-goodjob/wiki/design>

常见问题
---------------------
<https://github.com/wacai/pt-goodjob/wiki/FAQ>



联系我们
---------------------
#####  后台问题:xuanwu@wacai.com
#####  前端问题:baojun@wacai.com
#####  脚本问题:kaijin@wacai.com
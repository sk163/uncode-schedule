# uncode-schedule

>我对原作者开源的框架做了一部分的修改,包括bug的修复和新功能的添加.
1. 修改注册在zk上的server的code为可配置.
2. 修改ip黑名单配置可起作用.
3. 使spring/quartz task 也能将任务信息存储在zk上,这样管理界面就能看到具体的任务信息，并且区分任务类型,添加delay的spring task类型.
4. 修改任务的period和delay的时间单位为毫秒.
5. 添加手动执行任务的功能,并且可传参数.

*以下是框架的原文说明*


基于zookeeper的分布式任务调度组件，非常小巧，使用简单，只需要引入jar包，不需要单独部署服务端。确保所有任务在集群中不重复，不遗漏的执行。支持动态添加和删除任务。


# 功能概述

1. 基于zookeeper+spring task/quartz的分布任务调度系统。
2. 确保每个任务在集群中不同节点上不重复的执行。
3. 单个任务节点故障时自动转移到其他任务节点继续执行。
4. 任务节点启动时必须保证zookeeper可用，任务节点运行期zookeeper集群不可用时任务节点保持可用前状态运行，zookeeper集群恢复正常运期。
5. 支持动态添加和删除任务。
6. 添加ip黑名单，过滤不需要执行任务的节点。
7. 简单管理后台


说明：
* 单节点故障时需要业务保障数据完整性或幂等性
* 具体使用方式和spring task相同k


------------------------------------------------------------------------

# 模块架构

![模块架构](http://git.oschina.net/uploads/images/2016/0513/180808_6a6c1046_277761.png "模块架构")
![Worker构成](http://git.oschina.net/uploads/images/2016/0513/180912_8c9a24ec_277761.png "Worker构成")



------------------------------------------------------------------------

# Uncode-Schedule

## Spring bean

	public class SimpleTask {

		private static int i = 0;
		
		public void print() {
			System.out.println("===========start!=========");
			System.out.println("I:"+i);i++;
			System.out.println("=========== end !=========");
		}
	}

## xml配置

	<!-- 分布式任务管理器 -->
	<bean id="zkScheduleManager" class="cn.uncode.schedule.ZKScheduleManager"
		init-method="init">
		<property name="zkConfig">
			   <map>
				  <entry key="zkConnectString" value="127.0.0.1:2181" />
				  <entry key="rootPath" value="/uncode/schedule" />
				  <entry key="zkSessionTimeout" value="60000" />
				  <entry key="userName" value="ScheduleAdmin" />
				  <entry key="password" value="password" />
				  <entry key="isCheckParentPath" value="true" />
				  <entry key="ipBlacklist" value="127.0.0.2,127.0.0.3" />
			   </map>
		</property>
	</bean>
	
## API

1 动态添加任务

ConsoleManager.addScheduleTask(TaskDefine taskDefine);

2 动态删除任务

ConsoleManager.delScheduleTask(String targetBean, String targetMethod);

3 查询任务列表

ConsoleManager.queryScheduleTask();

------------------------------------------------------------------------

# 基于Spring Task的XML配置

## XML方式

1 Spring bean

	public class SimpleTask {

		private static int i = 0;
		
		public void print() {
			System.out.println("===========start!=========");
			System.out.println("I:"+i);i++;
			System.out.println("=========== end !=========");
		}
	}

2 xml配置

	<!-- 分布式任务管理器 -->
	<bean id="zkScheduleManager" class="cn.uncode.schedule.ZKScheduleManager"
		init-method="init">
		<property name="zkConfig">
			   <map>
				  <entry key="zkConnectString" value="127.0.0.1:2181" />
				  <entry key="rootPath" value="/uncode/schedule" />
				  <entry key="zkSessionTimeout" value="60000" />
				  <entry key="userName" value="ScheduleAdmin" />
				  <entry key="password" value="password" />
				  <entry key="isCheckParentPath" value="true" />
				  <entry key="ipBlacklist" value="127.0.0.2,127.0.0.3" />
			   </map>
		</property>
	</bean>
	<!-- Spring bean配置 -->
	<bean id="taskObj" class="cn.uncode.schedule.SimpleTask"/>
	<!-- Spring task配置 -->
	<task:scheduled-tasks scheduler="zkScheduleManager">
		<task:scheduled ref="taskObj" method="print"  fixed-rate="5000"/>
	</task:scheduled-tasks>
	
------------------------------------------------------------------------

## Annotation方式

1 Spring bean

	@Component
	public class SimpleTask {

		private static int i = 0;
		
		@Scheduled(fixedDelay = 1000) 
		public void print() {
			System.out.println("===========start!=========");
			System.out.println("I:"+i);i++;
			System.out.println("=========== end !=========");
		}
		
	}

2 xml配置

	<!-- 配置注解扫描 -->
    <context:annotation-config />
	<!-- 自动扫描的包名 -->
    <context:component-scan base-package="cn.uncode.schedule" />
	<!-- 分布式任务管理器 -->
	<bean id="zkScheduleManager" class="cn.uncode.schedule.ZKScheduleManager"
		init-method="init">
		<property name="zkConfig">
			   <map>
				  <entry key="zkConnectString" value="127.0.0.1:2181" />
				  <entry key="rootPath" value="/uncode/schedule" />
				  <entry key="zkSessionTimeout" value="60000" />
				  <entry key="userName" value="ScheduleAdmin" />
				  <entry key="password" value="password" />
				  <entry key="isCheckParentPath" value="true" />
				  <entry key="ipBlacklist" value="127.0.0.2,127.0.0.3" />
			   </map>
		</property>
	</bean>
	<!-- Spring定时器注解开关-->
	<task:annotation-driven scheduler="zkScheduleManager" />
	
------------------------------------------------------------------------

# 基于Quartz的XML配置

	注意：spring的MethodInvokingJobDetailFactoryBean改成cn.uncode.schedule.quartz.MethodInvokingJobDetailFactoryBean
	
	<bean id="zkScheduleManager" class="cn.uncode.schedule.ZKScheduleManager"
			init-method="init">
		<property name="zkConfig">
			   <map>
				  <entry key="zkConnectString" value="183.131.76.147:2181" />
				  <entry key="rootPath" value="/uncode/schedule" />
				  <entry key="zkSessionTimeout" value="60000" />
				  <entry key="userName" value="ScheduleAdmin" />
				  <entry key="password" value="password" />
				  <entry key="autoRegisterTask" value="true" />
				  <entry key="ipBlacklist" value="127.0.0.2,127.0.0.3" />
			   </map>
		</property>
	</bean>	


	<bean id="taskObj" class="cn.uncode.schedule.SimpleTask"/>

	<!-- 定义调用对象和调用对象的方法 -->
	<bean id="jobtask" class="cn.uncode.schedule.quartz.MethodInvokingJobDetailFactoryBean">
		<!-- 调用的类 -->
		<property name="targetObject" ref="taskObj" />
		<!-- 调用类中的方法 -->
		<property name="targetMethod" value="print" />
	</bean>
	<!-- 定义触发时间 -->
	<bean id="doTime" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
		<property name="jobDetail">
			<ref bean="jobtask"/>
		</property>
		<!-- cron表达式 -->
		<property name="cronExpression">
			<value>0/3 * * * * ?</value>
		</property>
	</bean>
	<!-- 总管理类 如果将lazy-init='false'那么容器启动就会执行调度程序  -->
	<bean id="startQuertz" lazy-init="false" autowire="no" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
		<property name="triggers">
			<list>
				<ref bean="doTime"/>
			</list>
		</property>
	</bean>

------------------------------------------------------------------------	
	
# uncode-schedule管理后台

访问URL：项目名称/uncode/schedule，如果servlet3.x以下，请手动配置web.xml文件
```
<servlet>
    <servlet-name>UncodeSchedule</servlet-name>
    <servlet-class>cn.uncode.schedule.web.ManagerServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>UncodeSchedule</servlet-name>
    <url-pattern>/uncode/schedule</url-pattern>
</servlet-mapping>
```

![img1](http://git.oschina.net/uploads/images/2016/0811/161906_b5720cac_277761.png "img1")
![img2](http://git.oschina.net/uploads/images/2016/0512/162217_0043832a_277761.png)
	
------------------------------------------------------------------------

# 大家都在使用uncode-schedule

- [快速递](http://www.ksudi.com)
- [优酷](http://www.youku.com/)
- [更多](https://git.oschina.net/uncode/uncode-schedule/issues/3)
	
------------------------------------------------------------------------

# 关于

作者：冶卫军（ywj_316@qq.com,微信:yeweijun）

技术支持QQ群：47306892

Copyright 2013 www.uncode.cn
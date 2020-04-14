# katyusha.quart demo

## 1. HOW TO

1.1 import katyusha.dependencies in dependencyManagement node in pom.xml

```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>katyusha</groupId>
                <artifactId>dependencies</artifactId>
                <version>0.0.2</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

1.2 import quart.job and rabbitmq.support in dependency in pom.xml

``` xml
 <dependency>
    <groupId>katyusha</groupId>
    <artifactId>quart.job</artifactId>
    <version>0.0.2-SNAPSHOT</version>
</dependency>
<dependency>
    <groupId>katyusha</groupId>
    <artifactId>rabbitmq.support</artifactId>
    <version>0.0.2-SNAPSHOT</version>
</dependency>

```

1.3 extends JobDistributor<T>

```java
package katyusha;
@component
public class DemoJobDetail extends JobDistributor<KatyushaTrigger>
```

@component **MUST** be set

1.4 implement the two method getRange and doJobDetail

```java
@Override protected DistributeRange getRange(@NonNull JobCommand<KatyushaTrigger> msg)
{...}

@Override protected void doJobDetail(JobDistributeMsg msg)
{...}
```

1.4 config your application.properties
some config ***MUST*** be set:

1.4.1 redis config

``` txt
spring.redis.pool.max-active=1000
spring.redis.pool.max-idle=8
spring.redis.pool.max-wait=300
spring.redis.testOnBorrow=true
spring.redis.timeout=3000
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=redis_password
```

1.4.2 rabbitmq config

```txt
spring.rabbitmq.addresses=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.virtual-host=/
spring.rabbitmq.publisher-confirms=true
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.listener.simple.concurrency=20
```

***spring.rabbitmq.listener.simple.concurrency=20*** is mean ~~***SHARE***~~ how much threads consume the messages on ***ALL detai queue*** 

1.4.3 zookeeper config(**which enhance in [release 0.0.4](#release-004-changelog)**)

```txt
curator.zookeeper.connectString=127.0.0.1:2181
curator.zookeeper.retryTimes=10
curator.zookeeper.sleepMsBetweenRetries=200
```

1.4.4 config katyusha trigger datasource

```properties
spring.datasource.katyusha.quart.driver-class-name=net.sf.log4jdbc.DriverSpy
spring.datasource.katyusha.quart.url=jdbc:log4jdbc:mysql://localhost:3306/katyushatrigger?useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true
spring.datasource.katyusha.quart.username=root
spring.datasource.katyusha.quart.password=123456
```

1.4.5 config your consumer of mq(**which enhance in [release 0.0.4](#release-004-changelog)**)

```txt
#katyusha mq config
katyusha.job.mq.config.queue[0].queueName=trigger_queue_name
katyusha.job.mq.config.queue[0].messageHandler=katyusha.DemoJobDetail
katyusha.job.mq.config.queue[0].exchange=trigger_exchange_name
katyusha.job.mq.config.queue[0].routeKey=somekeylike*
katyusha.job.mq.config.queue[0].threadSize=3

#katyusha mq detail config
katyusha.job.mq.detail.queue[0].queueName=detail_job_queue_name
katyusha.job.mq.detail.queue[0].messageHandler=katyusha.DemoJobDetail
katyusha.job.mq.detail.queue[0].exchange=detail_job_exchange_name
katyusha.job.mq.detail.queue[0].routeKey=detail*
katyusha.job.mq.detail.queue[0].threadSize=4
```
 ** ***For now the exchange,routeKey,threadSize is not used*** **

queueName must equals the rabbitmq queues name and same as in trigger database are.(**which enhance in [release 0.0.4](#release-004-changelog)**)

like below:

![avatar](https://gitee.com/tain198127/katyusha.quart.README/raw/master/WechatIMG244.png)

1.5 config database

Use ***katyusha.trigger.jar*** to edit trigger plan.  
The main point is TRIGGER_GROUP_NAME,TRIGGER_NAME,JOB_GROUP_NAME,JOB_NAME columns in KATYUSHA_TRIGGER table.
Compare with rabbitmq, TRIGGER_GROUP_NAME means ***exchange*** in rabbitmq, and TRIGGER_NAME means ***queue***, but its not used for now.
JOB_GROUP_NAME means ***exchange*** and JOB_NAME means ***queue*** in rabbitmq
For more detail please pay close attention to [katyusha.quart project](https://gitee.com/tain198127/katyusha.quart) or sent email to <tain198127@gmail.com>.

1.6 config rabbitmq

JUST like above, need add the exchange and queue, and bind them togeter. For more detail please visit [rabbitmq](http://www.rabbitmq.com/)

## 2. SCENARIO

It is ***FIT*** for you if you face those problems below:

+ Batching processes a large of data
+ Need more efficient on one task
+ HARDWARE resource and SYSTEM resource are ***NOT*** exhaust which including but not limited IOPS,MEM,CPU,TTL

LIKE: Clearning settlement in BANKING system or INTEREST ACCOUNTING SYSTEM

AND it is ***NOT FIT*** for you maybe， if you have sort of restrictions below:

+ The data without ***INTERGER PRIMARY KEY*** and it  ***CAN NOT BE HASHLIZATION***
+ The logic of batching process ***NOT stable***, need modify very quickly.
+ The interval is very shot, such like 1 second or 3 second.

LIKE: Geo data calculate and analysing or flash sale in e-commerce

## 3. ENVIRONMENT

### 3.1 HARDWARE REQUIREMENT

At least 2unit 4U 8G

At lease 100Mbps intranet

### 3.2 SYSTEM REQUIREMENT

CENTOS/UBUNTU/MACOSX/WINDOWS/Other Linux/UNIX

### 3.3 SOFTWARE REQUIREMENT

+ JDK 1.8

+ MySql 5.7

+ RabbitMQ 3.7.7

+ Redis 3.0.7

+ Zookeeper 3.4.7

+ Tomcat 8.5.20

### 3.4 Component REQUIREMENT

+ Spring 4.3.11
+ Spring boot 1.5.7


## release 0.0.4 changelog

### 1. Add annotation config for trigger and job and support application.properties config as well

code like below:

```java
@DistJobConfig(
        triggerGroupName = "trigger_exchange_name",
        triggerName = "trigger_queue_name",
        jobGroupName = "detail_job_exchange_name",
        jobName = "detail_job_queue_name"
    )
public class DemoJobDetail extends JobDistributor<KatyushaTrigger> {
}

```

or

```java
@DistJobConfig(
    jobGroupName = "detail_job_exchange_name1",
    jobName = "detail_job_queue_name1"
)
public class DemoJobDetail1 extends JobDistributor<KatyushaTrigger> {
}
```

**triggerGroupName** and **triggerName** means the exchange and queue in mq which hold message from **trigger** , both of them default value is ***EMPTY***, which means you don't have to config trigger consumer in every class. And
**jobGroupName** and **jobName** means the detail job, and you **MUST** do.

### 2. Add three type of checker for the queue config, which check cache, mq and db status

### 3. Dynamic distributor implements on loading time

Default implements is **REDIS**, if you do, you don't need write any extra code.
And on the other hand, if you want use **ZOOKEEPER**, please add **annotation** on your startup class, like below

``` java
@ImplicitDistributor(value = DistributorType.ZOOKEEPER)
public class App
{
    public static void main( String[] args )
    {
        SpringApplication.run(App.class, args);
    }

}

```

And add a config in your application.properties, code like below:

``` txt
curator.zookeeper.connectString=127.0.0.1:2181
```

which indicate zookeeper connect address, and other extra config have default value

### 4. Open some config property for Curator in application.properties

``` txt
curator.zookeeper.sessionTimeoutMs=1000
curator.zookeeper.connectionTimeoutMs=2000
```

which are ***ONLY*** work on **@ImplicitDistributor** annotation was marked and value is **value = DistributorType.ZOOKEEPER**. But if you code in you config and use **REDIS** implement, you will get no errors or any warning.

### 5. Some component version changed

Use zookeeper 3.4.8 ,curator 2.12.0 and use DistributedBarrier as distributed lock implements

### 6. DataBase changed

Add unique index in table katyusha_job and katyusha_trigger for prevent duplicate job_group_name and job_name.
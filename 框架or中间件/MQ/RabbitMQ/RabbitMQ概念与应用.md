## 1. 安装
1. 安装`Erlang26`，配置环境变量`%ERLANG_HOME%\bin`
2. 安装`RabbitMQ3.13`，进入`rabbitmq/sbin`并开启管理员`CMD`控制台，执行`rabbitmq-plugins enable rabbitmq_management`
3. 设置`rabbitmq`服务
## 2. MQ工作类型
- 简单模式
<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\simple简单模式.png" style="width:300px;height:100px;" />
- work工作模式
<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\work工作模式.png" style="width:300px;height:100px;" />
- 发布订阅共享模式
<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\发布订阅共享模式.png" style="width:300px;height:100px;" />
- 路由模式
<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\路由模式.png" style="width:300px;height:100px;" />
- 主题模式
<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\主题模式.png" style="width:300px;height:100px;" />

## 3. MQ四大概念
<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\MQ工作原理.png" style="width:700px;height:300px;" />

- 生产者：产生数据发送消息的程序
- 消费者：消费与接收含义相似，消费者类似于等待接收消息的程序
- 交换机：一方面接收来自生产者的消息，另一方面把接收到的消息推送到队列中
- 队列：消息缓冲区，消息只能存储在队列中，多个生产者可以将消息发送到一个队列中，多个消费者可以尝试从一个队列中取出消息

## 4. 简单使用
```java
package cn.buyforyou;

import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.Channel;  
import com.rabbitmq.client.Connection;  
import com.rabbitmq.client.ConnectionFactory;  
  
public class Send  
{  
    //队列名称  
	private final static String QUEUE_NAME = "hello";  
	
	public static void main(String[] argv)
    {
		//创建连接工厂
		ConnectionFactory factory = new ConnectionFactory();  
		factory.setHost("localhost");  
		factory.setUsername("guest");
		factory.setPassword("guest");
		
		try {
			Connection connection = factory.newConnection();
			Channel channel = connection.createChannel();
			/**
			 * 声明一个队列
			 * 1. 队列名称
			 * 2. 队列里的消息是否持久化 默认存储在内存中
			 * 3. 队列是否只供一个消费者进行消费 是否共享给多个消费者消费
			 * 4. 是否在最后一个消费者断开连接后队列自动删除
			 * 5. 其他参数
			 */
			channel.queueDeclare(QUEUE_NAME, false, false, false, null);
			String message = "hello world";
			/**
			 * 发送一个消息到队列
			 * 1. 发送到哪个交换机
			 * 2. 路由key，路由到哪个队列
			 * 3. 其他参数
			 * 4. 发送消息的消息体
			 */
			channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
			System.out.println("消息发送完毕");
			channel.close();  
			connection.close();
		} catch(Exception ex) {
			ex.printStackTrace();
		}
     }  
}  
```

```java
package cn.buyforyou;

import com.rabbitmq.client.Channel;  
import com.rabbitmq.client.Connection;  
import com.rabbitmq.client.ConnectionFactory;  

public class Recv {

    // 队列名称
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
	    //创建连接工厂
		ConnectionFactory factory = new ConnectionFactory();  
		factory.setHost("localhost");  
		factory.setUsername("guest");
		factory.setPassword("guest");
		
		try {
			Connection connection = factory.newConnection();
			Channel channel = connection.createChannel();
			
			DeliverCallback deliverCallback = (consumerTag,message) -> {
				System.out.println(new String(message.getBody()));
			}
			
			CancelCallback cancelCallback = consumerTag -> {
				System.out.println("消费被中断");
			}
			
			/**
			 * 1. 消费哪个队列
			 * 2. 消费成功后是否需要自动应答
			 * 3. 消息发送过来的回调
			 * 4. 被取消消费的回调
			 */
			channel.basicConsume(QUEUE_NAME,true,deliverCallback，cancelCallback);
		} catch(Exception) {
			ex.printStackTrace();
		}
    }
}
```

## 5. Work工作队列
**多线程消费者订阅队列**
```java
public class RabbitMQUtils {

	public static Channel getChannel() throw Excetion {
		ConnectionFactory factory = new ConnectionFactory();  
		factory.setHost("localhost");  
		factory.setUsername("guest");
		factory.setPassword("guest");
		
		Connection connection = factory.newConnection();
		Channel channel = connection.createChannel();
		return channel;
	}
}
```

```java
	RabbitMQUtils.getChannel();
	DeliverCallback deliverCallback = (consumerTag,message) -> {
		System.out.println(new String(message.getBody()));
	}
	
	CancelCallback cancelCallback = consumerTag -> {
		System.out.println("消费被中断");
	}
	
	/**
	 * 1. 消费哪个队列
	 * 2. 消费成功后是否需要自动应答
	 * 3. 消息发送过来的回调
	 * 4. 被取消消费的回调
	 */
	channel.basicConsume(QUEUE_NAME,true,deliverCallback，cancelCallback);
```

```java
	RabbitMQUtils.getChannel();
	/**
	 * 声明一个队列
	 * 1. 队列名称
	 * 2. 队列里的消息是否持久化 默认存储在内存中
	 * 3. 队列是否只供一个消费者进行消费 是否共享给多个消费者消费
	 * 4. 是否在最后一个消费者断开连接后队列自动删除
	 * 5. 其他参数
	 */
	channel.queueDeclare(QUEUE_NAME, false, false, false, null);
	
	Scanner scanner = new Scanner(System.in);
	while(scanner.hasNext()) {
		String message = scanner.next();
		/**
		 * 发送一个消息到队列
		 * 1. 发送到哪个交换机
		 * 2. 路由key，路由到哪个队列
		 * 3. 其他参数
		 * 4. 发送消息的消息体
		 */
		channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
		System.out.println("消息发送完毕");
	}
	channel.close();  
	connection.close();
```

## 6. 消息应答
消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成了部分突然消费者挂掉，为了保证消息在发送过程中不丢失，`rabbitmq`引入消息应答机制，消息应答就是：消费者在接收到消息并且处理该消息之后，告诉`rabbitmq`消费者处理完成，`rabbitmq`给消息标记删除记录，并把消息删除

- 自动应答：这种模式会在系统在**高吞吐量**与**数据传输安全**两方面做权衡，适用于消费者在某种速率下高效处理消息的情况使用
- 手动应答：适用于复杂业务，可以批量应答一个信道上的若干消息并减少网络阻塞（但不建议批量，因为应答的消息不一定已经业务上处理成功）

手动消息应答方式
- `channel.basicAck`肯定确认，并丢弃消息，存在批量应答的开关
- `channel.basicNack`否定确认
- `channel.basicReject`否定确认

如果消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失，导致消息未发送 ACK 确认，RabbitMQ 将了解到消息未完全处理，并将对其重新排队。如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息

```java
	RabbitMQUtils.getChannel();
	DeliverCallback deliverCallback = (consumerTag,message) -> {
		System.out.println(new String(message.getBody()));
		SleepUtils.sleep(10);
		channel.basicAck(message.getEnvelope().getDeliveryTag(), false);//不批量
	}
	
	CancelCallback cancelCallback = consumerTag -> {
		System.out.println("消费被中断");
	}
	
	/**
	 * 1. 消费哪个队列
	 * 2. 消费成功后是否需要自动应答
	 * 3. 消息发送成功后的回调
	 * 4. 被取消消费的回调
	 */
	boolean autoAck = false;//手动应答
	channel.basicConsume(QUEUE_NAME,autoAck,deliverCallback，cancelCallback);
```

## 7. 队列与消息持久化
默认情况下`RabbitMQ`退出或由于某种原因崩溃时，它忽视队列自动应答和消息，即消息队列在内存中丢失
确保消息不会在内存丢失需要做两件事：我们需要将队列和消息都标记为持久化

### 1. 队列持久化

*当队列开启持久化时，重启MQ后队列依然存在。如果之前声明的队列不是持久化的，需要把原先队列删除，或者重新新建一个持久化队列*
```java
//生产者channel绑定消息队列时声明持久化
/**
 * 声明一个队列
 * 1. 队列名称
 * 2. 队列里的消息是否持久化 默认存储在内存中
 * 3. 队列是否只供一个消费者进行消费 是否共享给多个消费者消费
 * 4. 是否在最后一个消费者断开连接后队列自动删除
 * 5. 其他参数
 */
channel.queueDeclare(QUEUE_NAME, true, false, false, null);
```

### 2. 消息持久化
*当消息开启持久化时，重启MQ后消息依然存在*
```java
/**
 * 发送一个消息到队列
 * 1. 发送到哪个交换机
 * 2. 路由key，路由到哪个队列
 * 3. 其他参数
 * 4. 发送消息的消息体
 */
 //设置消息属性MessageProperties.PERSISTENT_TEXT_PLAIN弱持久化，存在消息由缓存到持久的入磁盘前的时间间隙
channel.basicPublish("",QUEUE_NAME,MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes());
```

### 3. 发布确认
MQ持久化包括三步：
1. 队列持久化
2. 消息持久化
3. 发布确认，即消息和队列真正持久化到磁盘上后，MQ返回`ack`信号给生产者

**开启生产者的发布确认：`channel.confirmSelect()`**
- 单个确认`channel.waitForConfirms()、waitForConfirmsOrDie(long)`：同步确认发布，发布一个消息后被确认才能继续发布后续消息，指定时间内消息没被确认则抛出异常
```java
	channel.queueDeclare(queueName, true, false, false, null);
	//开启发布确认
	channel.confirmSelect();
	...
	channel.basicPublish("", queueName, null, message.getBytes());
	//服务端返回发布确认结果
	boolean resultFlag = channel.waitForConfirms();
	if(resultFlag == true) {
		...
	}
	...
```
- 批量确认`channel.basicPublish("", queueName, null, message.getBytes())+channel.waitForConfirms()`：同步批量确认发布，发声发布错误无法确认具体哪条消息发生错误
```java
	channel.queueDeclare(queueName, true, false, false, null);
	//开启发布确认
	channel.confirmSelect();
	...
	for(...) {
		channel.basicPublish("", queueName, null, message.getBytes());
	}
	channel.waitForConfirms();
	...
```
- 异步批量确认``：
<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\异步批量确认.png" style="width:700px;height:350px;" />
```java
	channel.queueDeclare(queueName, true, false, false, null);
	//开启发布确认
	channel.confirmSelect();
	//监听消息确认成功的回调
	ConfirmCallback ackCallback = (deliveryTag,multiple) -> {
		System.out.println("确认的消息" + deliveryTag);
	};
	//监听消息确认失败的回调
	ConfirmCallback nackCallback = (deliveryTag,multiple) -> {
		System.out.println("未确认的消息" + deliveryTag);
	};
	
	//准备消息的监听器
	channel.addConfirmListener(ackCallback,nackCallback);
	...
	for(...) {
		channel.basicPublish("", queueName, null, message.getBytes());
	}
```

**异步发布确认的未确认消息处理**：使用`ConcurrentLinkedQueue`存储内存中的未确认消息，把消息从监听线程传递给发布线程
```java
	channel.queueDeclare(queueName, true, false, false, null);
	//开启发布确认
	channel.confirmSelect();
	
	//记录全量消息
	ConcurrentSkipListMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();
	
	//监听消息确认成功的回调
	ConfirmCallback ackCallback = (deliveryTag,multiple) -> {
		System.out.println("确认的消息" + deliveryTag);
		//记录确认成功的消息并删除
		if(multiple) {
			ConcurrentNavigableMap<Long, String> confirmed = outstandingConfirms.headMap(deliveryTag);
			confirmed.clear();
		} else {
			outstandingConfirms.remove(deliveryTag);
		}
	};
	//监听消息确认失败的回调
	ConfirmCallback nackCallback = (deliveryTag,multiple) -> {
		String message = outstandingConfirms.get(deliveryTag);
		System.out.println("未确认的消息与tag" + message + " " + deliveryTag);
		//System.out.println("未确认的消息" + deliveryTag);
	};
	
	//准备消息的监听器
	channel.addConfirmListener(ackCallback,nackCallback);
	...
	for(...) {
		channel.basicPublish("", queueName, null, message.getBytes());
		//记录全量消息
		outstandingConfirms.put(channel.getNextPublishSeqNo(), message);
	}
```

## 8. 消息分发与预取值
默认消息公平轮询分发，可以设置为不公平分发，消费者平均消费小心，通过**设置消费者的信道属性**`channel.basicQos(1)`
默认轮询情况下预取为1；可以指定消息分发的时候，给消费者的预取值`prefetch：channel.basicQos(prefetch)`

`channel.basicQos(prefetch)`可以理解为指定消费者能最多处理积压消费的消息个数

## 9. 交换机
`RabbitMQ`消息传递模型的核心思想是：生产者生产的消息从不会直接发送到队列。实际上，通常生产者甚至都不知道这些消息传递传递到了哪些队列中
相反，生产者只能将消息发送到交换机`exchange`，交换机工作的内容非常简单，一方面它接收来自生产者的消息，另一方面将它们推入队列。交换机必须确切知道如何处理收到的消息：是应该把这些消息放到特定队列还是说把他们到许多队列中还是说应该丢弃它们，这都由交换机的类型来决定

交换机把消息路由到具体某个队列需要`routingkey`来匹配路由到正确队列，此为交换机与队列的绑定`binding`

### 1. 无名exchange
`channel.basicPublish("",QUEUE_NAME,null,message.getBytes())`，第一个参数就是交换机的名称
第二个参数在没有指定交换机时可以直接指定为队列名称，如果指定了交换机后需要使用`routingKey(bindingkey)`绑定

**临时队列**：连接`RabbitMQ`时被`RabbitMQ Server`创建的一个全新的、具有随机名称的队列，一旦断开连接队列则自动被删除`String queueName = channel.queueDeclare().getQueue()`

### 2. Fanout扇出广播Exchange
<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\fanout广播.png" style="width:500px;height:200px;" />

将消息广播到所路由到的所有队列，**`fanout`类型`exchange`会把同一个消息从交换机发给匹配到的所有队列**
```java
	public static final String EXANGE_NAME = "logs";
	
	Channel channel = RabbitMqUtils.getChannel();
	//声明一个交换机
	channel.exchangeDeclare(EXCHANGE_NAME, "FANOUT");
	//生成一个临时队列
	String queueName = channel.queueDeclare().getQueue();
	//绑定交换机与队列，第三个参数为routingKey
	channel.queueBind(queueName, EXCHANGE_NAME, "");
	
	//开始接收消息
	...
	
	channel.basicConsume(queueName,true,deliverCallback，cancelCallback);
	
```

```java
	Channel channel = RabbitMqUtils.getChannel();
	Scanner sc = new Scanner(System.in);
	while(sc.hasNext()) {
		String message = sc.nextLine();
		channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
		System.out.println("生产者发出消息" + message);
	}
```

```java
private final static String FANOUT_EXCHANGE_NAME = "fanout-exchange";
private final static String FANOUT_QUEUE1_NAME = "fanout-queue1";
private final static String FANOUT_QUEUE2_NAME = "fanout-queue2";
private final static String FANOUT_ROUTING_KEY = "";

//Produce
ConnectionFactory factory = new ConnectionFactory();  
factory.setHost("localhost");  
factory.setPort(5672);  
factory.setUsername("guest");  
factory.setPassword("guest");  
  
Connection connection = factory.newConnection();  
Channel channel = connection.createChannel();  
  
channel.exchangeDeclare(FANOUT_EXCHANGE_NAME, BuiltinExchangeType.FANOUT);  
channel.queueDeclare(FANOUT_QUEUE1_NAME, true, false, false, null);  
channel.queueDeclare(FANOUT_QUEUE2_NAME, true, false, false, null);  
channel.queueBind(FANOUT_QUEUE1_NAME, FANOUT_EXCHANGE_NAME, FANOUT_ROUTING_KEY);  
channel.queueBind(FANOUT_QUEUE2_NAME, FANOUT_EXCHANGE_NAME, FANOUT_ROUTING_KEY);  
  
channel.basicPublish(FANOUT_EXCHANGE_NAME, FANOUT_ROUTING_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN, getRandomString(10).getBytes());

//Consume
ConnectionFactory factory = new ConnectionFactory();  
factory.setHost("localhost");  
factory.setPort(5672);  
factory.setUsername("guest");  
factory.setPassword("guest");  
  
Connection connection = factory.newConnection();  
Channel channel = connection.createChannel();  
  
DeliverCallback deliverCallback = (consumerTag, message) -> {  
    log.info(consumerTag);  
    log.info(new String(message.getBody()));  
    log.info("消费成功");  
};  
  
CancelCallback cancelCallback = consumerTag -> {  
    log.info(consumerTag);  
    log.info("消费被中断");  
};  
  
channel.basicConsume(FANOUT_QUEUE1_NAME, true, deliverCallback, cancelCallback);  
channel.basicConsume(FANOUT_QUEUE2_NAME, true, deliverCallback, cancelCallback);
```
### 3. Direct直接路由Exchange
<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\direct单个绑定.png" style="width:500px;height:200px;" />
<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\direct多个绑定.png" style="width:500px;height:200px;" />


支持多重`routingKey`绑定
```java
	public static final String EXANGE_NAME = "console";
	
	Channel channel = RabbitMqUtils.getChannel();
	//声明一个交换机
	channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
	//声明一个队列
	channel.queueDeclare("console", false, false, false, null);
	//绑定交换机与队列，第三个参数为routingKey
	channel.queueBind("console", EXANGE_NAME, "info");
	channel.queueBind("console", EXANGE_NAME, "error");
	
	//开始接收消息
	...
	
	channel.basicConsume(queueName,true,deliverCallback，cancelCallback);
```

```java
	Channel channel = RabbitMqUtils.getChannel();
	Scanner sc = new Scanner(System.in);
	while(sc.hasNext()) {
		String message = sc.nextLine();
		channel.basicPublish(EXCHANGE_NAME, "info", null, message.getBytes("UTF-8"));
		System.out.println("生产者发出消息" + message);
	}
```

### 4. Topic主题Exchange
<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\topic主题.png" style="width:500px;height:200px;" />


`topic`交换机的`routingKey`不能随便乱写，需要满足为单调列表，以点号分隔开，如`"stock.usd.nyse/nyse.vmw"`，存在替换符：星号`*`代替一个单词，井号`#`代替零个或多个单词，如：`*.orange.*/*.*.rabbit/lazy.#`

当`routingKey`被设置为单独一个`#`，则类似`fanout`；当`routingKey`中没有出现`#`和`*`，则类似`direct`

```java
	public static final String EXANGE_NAME = "topic_logs";
	
	Channel channel = RabbitMqUtils.getChannel();
	//声明一个交换机
	channel.exchangeDeclare(EXCHANGE_NAME, "topic");
	//声明一个队列
	channel.queueDeclare("Q1", false, false, false, null);
	//绑定交换机与队列，第三个参数为routingKey
	channel.queueBind("Q1", EXANGE_NAME, "*.orange.*");
	
	//开始接收消息
	...
	
	channel.basicConsume(queueName,true,deliverCallback，cancelCallback);
```

```java
	
	Map<String,String> bindingKeyMap = new HashMap<>();
	bindingKeyMap.put("quick.orange.rabbit","被队列Q1Q2接收到");
	bindingKeyMap.put("lazy.orange.elephant","被队列Q1Q2接收到");
	bindingKeyMap.put("quick.orange.fox","被队列Q1接收到");
	bindingKeyMap.put("lazy.brown.fox","被队列Q2接收到");
	bindingKeyMap.put("lazy.pink.rabbit","虽然满足两个绑定但只被队列Q2接收一次");
	bindingKeyMap.put("quick.brown.fox","不匹配任何绑定不会被任何队列接收到会被丢弃");
	bindingKeyMap.put("quick.orange.male.rabbit","是四个单词不匹配任何绑定会被丢弃");
	bindingKeyMap.put("lazy.orange.male.rabbit","是四个单词但匹配Q2"):
	
	Channel channel = RabbitMqUtils.getChannel();
	for(Map.Entry<String, String> bindingKeyEntry : bindingKeyMap.entrySet()) {
		String routingKey = bindingKeyEntry.getKey();
		String message = bindingKeyEntry.getValue();
		channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes("UTF-8"));
	}
```

## 10. 死信队列
生产者把消息投递到交换机或者队列中，消费者从队列取出消息进行消费，**出于某些原因导致队列中某些消息无法被正常消费**，这些消息会经过后续处理变成死信拉入死信队列（如：保证订单业务消息处理不丢失把消息投入死信队列，下单成功然而支付超时订单自动失效把消息投入死信队列）

死信来源：
1. 消息存在`TTL`过期时间
2. 队列达到最大长度，队列满了
3. 消息被拒绝`basic.reject`或者`basic.nack`，并且`requeue=false`

<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\死信队列.png" style="width:500px;height:500px;" />

```java
public class Consumer01 {
	//普通交换机
	public static final String NORMAL_EXCHANGE = "normal_exchange";
	//死信交换机
	public static final String DEAD_EXCHANGE = "dead_exchange";
	//普通队列的名称
	public static final String NORMAL_QUEUE = "normal_queue";
	//死信队列的名称
	public static final String DEAD_QUEUE = "dead_queue";
	
	public static void main(String[] main) {
		Channel channel = RabbitMqUtils.getChannel();
		
		Map<String, Object> arguments = new HashMap<>();
		//消息过期TTL，设置整个消息的过期时间
		arguments.put("x-message-ttl", 10000);
		//设置队列最大容量长度，数量超出的消息则进入死信队列
		arguments.put("x-max-length", 6);
		//正常队列设置死信交换机
		arguments.put("x-dead-letter-exchange", DEAD_EXCHANGE);
		//设置死信routingKey
		arguments.put("x-dead-letter-routing-key", "lisi");
		
		//声明普通交换机与死信交换机
		channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
		channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
		//声明普通队列与死信队列
		channel.queueDeclare(NORMAL_QUEUE, false, false, false, arguments);
		channel.queueDeclare(DEAD_QUEUE, false, false, false, null);
		//绑定交换机与队列
		channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "zhangsan");
		channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "lisi");
		
		DeliverCallback deliverCallback = (consumerTag, message) -> {
			System.out.println(new String(message.getBody(), "UTF-8"));
			//消息拒绝
			if(message.equals("info5")) {
				//拒绝消息 参数为消息的标签与取消放回队列
				channel.basicReject(message.getEnvelope().getDeliveryTag(), false);
			} else {
				//手动应答确认
				channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
			}
		};
		
		CancelCallback cancelCallback = consumerTag -> {
			System.out.println("消费被中断");
		};
		//false开启手动应答
		channel.basicConsume(NORMAL_QUEUE, false, deliverCallback, cancelCallback);
	}
}
```

```java
public class Producer {
	//普通交换机
	public static final String NORMAL_EXCHANGE = "normal_exchange";
	
	public static void main(String[] args) throws Exception {
		Channel channel = RabbitMqUtils.getChannel();
		//死信消息 设置TTL 单独设置一个消息的过期时间
		/*
		AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().expiration("10000").build();
		*/
		
		for(int i = 1; i < 11; i++) {
			String message = "info" + i;
			channel.basicPublish(NORMAL_EXCHANGE, "zhangsan", properties, message.getBytes());
		}
	}
}
```

## 11. TTL延迟队列
<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\TTL延迟队列.png" style="width:700px;height:300px;" />

### 1. 基于死信的延迟队列
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

```java
public class TtlQueueConfig {

	//普通交换机与死信交换机
	public static final String X_EXCHANGE = "X";
	public static final String Y_DEAD_LETTER_EXCHANGE = "Y";
	
	//普通延迟队列
	public static final String QUEUE_A = "QA";
	public static final String QUEUE_B = "QB";
	public static final String QUEUE_C = "QC";
	
	//死信队列
	public static final String DEAD_LETTER_QUEUE = "QD";
	
	@Bean
	public DirectExchange xExchange() {
		return new DirectExchange(X_EXCHANGE);
	}
	
	@Bean
	public DirectExchange yExchange() {
		return new DirectExchange(Y_DEAD_LETTER_EXCHANGE);
	}
	
	@Bean
	public Queue queueA() {
		Map<String, Object> arguments = new HashMap<String, Object>(3);
		arguments.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
		arguments.put("x-dead-letter-routing-key", "YD");
		//设置整个队列的TTL过期时间
		arguments.put("x-message-ttl", 10000);
		return QueueBuilder.durable(QUEUE_A).withArguments(arguments).build();
	}
	
	@Bean
	public Queue queueB() {
		Map<String, Object> arguments = new HashMap<String, Object>(3);
		arguments.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
		arguments.put("x-dead-letter-routing-key", "YD");
		//设置整个队列的TTL过期时间
		arguments.put("x-message-ttl", 40000);
		return QueueBuilder.durable(QUEUE_B).withArguments(arguments).build();
	}
	
	@Bean
	public Queue queueC() {
		Map<String, Object> arguments = new HashMap<String, Object>(2);
		arguments.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
		arguments.put("x-dead-letter-routing-key", "YD");
		return QueueBuilder.durable(QUEUE_C).withArguments(arguments).build();
	}
	
	@Bean
	public Queue queueD() {
		return QueueBuilder.durable(DEAD_LETTER_QUEUE).build();
	}
	
	@Bean
	public Binding queueABindingX(@Qualifier("queueA") Queue queueA, @Qualifier("xExchange") DirectExchange xExchange) {
		return BindingBuilder.bind(queueA).to(xExchange).with("XA");
	}
	
	@Bean
	public Binding queueBBindingX(@Qualifier("queueB") Queue queueB, @Qualifier("xExchange") DirectExchange xExchange) {
		return BindingBuilder.bind(queueB).to(xExchange).with("XB");
	}
	
	@Bean
	public Binding queueCBindingX(@Qualifier("queueC") Queue queueC, @Qualifier("xExchange") DirectExchange xExchange) {
		return BindingBuilder.bind(queueC).to(xExchange).with("XC");
	}
	
	@Bean
	public Binding queueABindingX(@Qualifier("queueD") Queue queueD, @Qualifier("yExchange") DirectExchange yExchange) {
		return BindingBuilder.bind(queueD).to(yExchange).with("YD");
	}
}
```

```java
@Slf4j
@RestController
@RequestMapping("/ttl")
public class SendMsgController {
	
	@Autowired
	private RabbitTemplate rabbitTemplate;
	
	@GetMapping("/sendMsg/{message}")
	public void sendMsg(@PathVariable String message) {
		log.info("当前时间: {}, 发送信息给两个TTL队列: {}", new Date().toString(), message);
		rabbitTemplate.convertAndSend("xExchange", "XA", message);
		rabbitTemplate.convertAndSend("xExchange", "XB", message);
	}

	@GetMapping("/sendExpirationMsg/{message}/{ttlTime}")
	public void sendExpirationMsg(@PathVariable String message, @PathVariable String ttlTime) {
		log.info("当前时间: {}, 发送信息给两个TTL队列: {}, 发送信息的过期时间: {}", new Date().toString(), message, ttlTime);
		//单独设置每个消息的TTL过期时间
		/**
		 * 如果在消息上设置TTL过期时间，消息并不会按时死亡，因为只会检查第一个消息是否过期，是则进入死信队列
		 * 如果第一个消息延迟时间过长，而第二个消息延迟时间很短，则第二个消息不会优先得到执行
		**/
		rabbitTemplate.convertAndSend("xExchange", "XC", message, msg -> {
			msg.getMessageProperties().setExpiration(ttlTime);
			return msg;
		});
	}
}
```

```java
@Slf4j
@Component
public class DeadLetterQueueConsumer {
	
	@RabbitListener(queues = "QD")
	public void receiveD(Message msg, Channel channel) throws Exception {
		String msg = new String(msg.getBody());
		log.info("当前时间:{}, 收到死信队列消息:{}", new Date().toString(), msg);
	}
}
```

### 2. 基于插件的交换机延迟队列
**针对死信的延迟队列进行优化**
安装`rabbit_delayed_message_exchange`插件，解压放置到`rabbitmq`的插件目录
执行命令`rabbitmq-plugins enable rabbitmq_delayed_message_exchange`

```java
@Configuration
public class DelayedQueueConfig {
	
	public static final String DELAYED_QUEUE_NAME = "delayed.queue";
	public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
	public static final String DELAYED_ROUTING_KEY = "delayed.routingKey";
	
	@Bean
	public CustomExchange delayedExchange() {
		Map<String, Object> arguments = new HashMap<String, Object>();
		arguments.put("x-delayed-type", "direct");
		/**
		 * 交换机的名称
		 * 交换机的类型
		 * 交换机是否持久化
		 * 是否需要自动删除
		 * 其他参数
		**/
		return new CustomExchange(DELAYED_EXCHANGE_NAME, "x-delayed-message", true, false, );
	}
	
	@Bean
	public Queue delayedQueue() {
		return new Queue(DELAYED_QUEUE_NAME);
	}
	
	@Bean
	public Binding delayedBinding(@Qualifier("delayedQueue") Queue delayedQueue, @Qualifier("delayedExchange") CustomExchange delayedExchange) {
		return BindingBuilder.bind(delayedQueue).to(delayedExchange).
			with(DELAYED_ROUTING_KEY).noargs();
	}
}
```

```java
@Slf4j
@RestController
@RequestMapping("/ttl")
public class SendMsgController {
	
	@Autowired
	private RabbitTemplate rabbitTemplate;
	
	@GetMapping("/sendDelayedMsg/{message}/{delayTime}")
	public void sendDelayedMsg(@PathVariable String message, @PathVariable String delayTime) {
		log.info("当前时间: {}, 发送信息: {}, TTL: {}", new Date().toString(), message, delayTime);
		rabbitTemplate.convertAndSend(DELAYED_EXCHANGE_NAME, DELAYED_ROUTING_KEY, 
			message, 
			msg -> {
				msg.getMessageProperties().setDelay(delayTime);
				return msg;
			});
	}
}
```

```java
@Slf4j
@Component
public class DelayQueueConsumer {
	
	@RabbitListener(queues = DELAYED_QUEUE_NAME)
	public void receiveDelayedMsg(Message msg) {
		String msg = new String(msg.getBody());
		log.info("当前时间:{}, 收到延迟队列消息:{}", new Date().toString(), msg);
	}
}
```

## 12. 发布确认高级
`rabbitmq`宕机造成生产者投递失败，消息丢失，需要手动处理和恢复

### 1. 回调接口
**只根据交换机是否接收到消息进行回调，不根据队列是否接收到消息决定**
```prop
spring.rabbitmq.host=182.92.234.71
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=123
spring.rabbitmq.publisher-confirm-type=none/correlated/simple
```

- `none`：禁用发布确认模式，为默认值
- `correlated`：发布消息成功到交换机后会触发回调
- `simple`：也会在交换机接收到消息后回调接口，但发布消息成功后会调用`waitForConfirms`或者`waitForConfirmsOrDie`等待`broker`返回发送结果

```java
@Sl4j
@Component
puclic class MyCallBack implements RabbitTemplate.ConfirmCallback {

	@Autowired
	private RabbitTemplate rabbitTemplate;
	
	@PostConstruct
	public void init() {
		rabbitTemplate.setConfirmCallback(this);
	}
	
	/**
	 * 交换机确认回调方法
	 * correlationData 保存回调消息的ID以及其他消息内容
	 * 交换机是否收到消息
	 * 消息失败的原因
	 */
	@Override
	public void confirm(CorrelationData correlationData, boolean b, String s) {
		
		//此处的correlationData使用的是ConfirmCallback的correlationData
		String id = correlationData != null? correlationData.getId(): "";
		if(ack) {
			log.info("成功");
		} else {
			log.info("失败");
		}
	}
}

@Controller
@RequestMapping("/produce")
public class produceController {
	
	@Autowired
	private RabbitTemplate rabbitTemplate;
	
	@GetMapping("/send/{msg}")
	public void sendMsg(@PathVariable String msg) {
		
		//此处的correlationData使用的是ConfirmCallback的correlationData
		CorrelationData correlationData = new CorrelationData("1");
		
		rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME, ConfirmConfig.CONFIRM_ROUTING_KEY, msg, correlationData);
		log.info("");
	}
}
```

### 2. 回退消息
**仅开启生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息，如果发现该消息不可路由，则消息会被直接丢弃**
**回退则是将消息返回给生产者**
```property
spring.rabbitmq.host=182.92.234.71
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=123
spring.rabbitmq.publisher-confirm-type=none/correlated/simple
spring.rabbitmq.publisher-returns=true
```

```java
@Sl4j
@Component
puclic class MyCallBack implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnCallback {

	@Autowired
	private RabbitTemplate rabbitTemplate;
	
	@PostConstruct
	public void init() {
		rabbitTemplate.setConfirmCallback(this);
		rabbitTemplate.setReturnCallback(this);
	}
	
	@Override
	public void confirm(CorrelationData correlationData, boolean b, String s) {
		...
	}
	
	//在消息传递过程中不可达目的地时将消息返回给生产者
	/**
	 * 消息
	 * 回退码
	 * 回退原因
	 * 回退交换机
	 * 被回退的routingKey
	 */
	@Override
	public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
		
	}
}

@Controller
@RequestMapping("/produce")
public class produceController {
	
	@Autowired
	private RabbitTemplate rabbitTemplate;
	
	@GetMapping("/send/{msg}")
	public void sendMsg(@PathVariable String msg) {
		
		//此处的correlationData使用的是ConfirmCallback的correlationData
		CorrelationData correlationData = new CorrelationData("1");
		
		rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME, ConfirmConfig.CONFIRM_ROUTING_KEY, msg, correlationData);
		log.info("");
	}
}
```

### 3. 备份交换机
无法被投递到目标交换机的消息，目标交换机会发送信号给备份交换机，备份交换机通过自己的路由和队列将消息发送给消费者

**达到备份与报警的功能**

<img src="D:\Project\IT-notes\框架or中间件\MQ\RabbitMQ\img\备份交换机.png" style="width:700px;height:250px;" />

```java
	//确认交换机
	public static final String CONFIRM_EXCHANGE_NAME = "confirm_exchange";
	
	//备份交换机
	public static final String BACKUP_EXCHANGE_NAME = "backup_exchange";
	
	//备份队列
	public static final String BACKUP_QUEUE_NAME = "backup_queue";
	
	//报警队列
	public static final String WARNING_QUEUE_NAME = "warning_queue";
	
	@Bean
	public DirectExchange confirmExchange() {
		return ExchangeBuilder.directExchange(CONFIRM_EXCHANGE_NAME)
			.durable(true)
			.withArgument("alternate-exchange", BACKUP_EXCHANGE_NAME) //设置备份交换机
			.build();
	}
	
	@Bean
	public DirectExchange backupExchange() {
		return new FanoutExchange(BACKUP_EXCHANGE_NAME);
	}
	
	@Bean
	public Queue backupQueue() {
		return QueueBuilder.durable(BACKUP_QUEUE_NAME).build();
	}
	
	@Bean
	public Queue warningQueue() {
		return QueueBuilder.durable(WARNING_QUEUE_NAME).build();
	}
	
	@Bean
	public Binding queueBackupBindingB(@Qualifier("backupQueue") Queue backupQueue, @Qualifier("backupExchange") DirectExchange backupExchange) {
		return BindingBuilder.bind(backupQueue).to(backupExchange);
	}
	
	@Bean
	public Binding queueWarningBindingB(@Qualifier("warningQueue") Queue warningQueue, @Qualifier("backupExchange") DirectExchange backupExchange) {
		return BindingBuilder.bind(warningQueue).to(backupExchange);
	}
	
```

## 13. 消息重复消费
**确保幂等性**
### 1. 唯一ID+指纹码机制

### 2. Redis SETNX原子性

## 14. 优先级队列
```java
//队列添加优先级
Map<String, Object> params = new HashMap<>();
params.put("x-max-priority", 10);
channel.queueDeclare("hello", true, false, false, params);

//消息添加优先级
AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();
```

## 15. 惰性队列
消息保存在磁盘中，存在default、lazy两种模式，应用场景是消息积压
```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-queue-mode", "lazy");
channel.queueDeclare("myqueue", false, false, false, args);
```

## 16. 集群
- 单机模式
- 普通集群模式：**多台机器**上启动**多个rabbitmq实例**，**每个机器启动一个。**但是你创建的**queue**，只会放在**一个rabbtimq实例**上，但是**每个实例都同步queue的元数据(存放含queue数据的真正实例位置)**。消费的时候，实际上如果连接到了另外一个实例，那么那个实例会从queue所在实例上拉取数据过来
- 镜像模式：创建的queue，无论元数据还是queue里的消息都会存在于多个实例上，然后每次你写消息到queue的时候，都会自动把消息到多个实例的queue里进行消息同步
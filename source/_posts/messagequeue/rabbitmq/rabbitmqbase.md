---
title: rabbitmq 基础
date: 2022-05-11 15:08:45
categories:
  - rabbitmq
  - queue
tags:
  - rabbitmq
  - queue
---

## 概要

RabbitMQ 是一个消息代理：它接受和转发消息。可以将其视为邮局：将要邮寄的邮件放入邮箱时，可以确定邮递员最终会将邮件投递给您的收件人。在这个类比中，RabbitMQ 是一个邮箱、一个邮局和一个邮递员。

RabbitMQ 和邮局之间的主要区别在于它不处理纸张，而是接受、存储和转发二进制数据块 -消息。

RabbitMQ 和一般的消息传递使用一些行话。

- 生产无非就是发送。发送消息的程序是**生产者**：
![image](/images/rabbitmq-base/producer.png)
- **队列**是位于 RabbitMQ 中的邮箱的名称。尽管消息流经 RabbitMQ 和应用程序，但它们只能存储在队列中。队列仅受主机的内存和磁盘限制，它本质上是一个大的消息缓冲区。许多生产者可以发送去一个队列的消息，许多消费者可以尝试从一个队列接收数据。这就是我们表示队列的方式：
![image](/images/rabbitmq-base/queue.png)
- 消费与接收具有相似的含义。**消费者**是一个主要等待接收消息的程序:
![image](/images/rabbitmq-base/consumer.png)
**请注意，生产者、消费者和代理不必驻留在同一主机上**；事实上，在大多数应用程序中它们都没有。应用程序也可以既是生产者又是消费者。

## 消息模式

### 简单模式

生产者直接将消息发送到队列，消费者从队列获取消息
![image](/images/rabbitmq-base/python-one.png)

#### 示例

发送消息：发布者将连接到 RabbitMQ，发送一条消息，然后退出。
![image](/images/rabbitmq-base/sending.png)

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.nio.charset.StandardCharsets;

public class Send {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        // 创建rabbitmq连接和通道
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            // 声明将要发送消息的队列
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message = "Hello World!";
            // 发送消息
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}
```

接收消息：监听队列并接收消息，接收一条消息后不自动退出
![images](/images/rabbitmq-base/receiving.png)

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;
import java.nio.charset.StandardCharsets;

public class Recv {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        // 创建连接和通道
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        // 声明接收消息的队列(与发送队列匹配)
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");
        // 缓存服务器推送给我们的消息
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" + message + "'");
        };
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
    }
}
```

### 工作队列模式

创建一个工作队列，用于在多个工作人员之间分配耗时的任务。
**工作队列（又名：任务队列）**背后的主要思想是避免立即执行资源密集型任务而不得不等待它完成。相反，可以将任务安排在以后完成。将任务封装 为消息并将其发送到队列。在后台运行的工作进程将弹出任务并最终执行作业。运行许多工作人员时，任务将在他们之间共享。
这个概念在 Web 应用程序中特别有用，在这些应用程序中，无法在短暂的 HTTP 请求窗口中处理复杂的任务。
![image](/images/rabbitmq-base/python-two.png)

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;
import java.nio.charset.StandardCharsets;

public class Recv {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        // 创建连接和通道
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        // 声明接收消息的队列(与发送队列匹配)
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");
        // 缓存服务器推送给我们的消息
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println(" [x] Received '" + message + "'");
            doWork(message);
        };
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
    }

    private void doWork(String task) {
      for (char c: task.toCharArray()) {
        if (c == '.') Thread.sleep(1000);
      }
    }
}

```

#### 循环调度

使用任务队列的优点之一是能够轻松并行工作。如果积压工作，可以添加更多的工作人员，这样就可以轻松扩展。
首先，尝试同时运行两个工作实例。他们都会从队列中获取消息，但究竟如何呢？让我们来看看。
默认情况下，RabbitMQ 会按顺序将每条消息发送给下一个消费者。平均而言，每个消费者都会收到相同数量的消息。这种分发消息的方式称为循环。

#### 消息确认

完成一项任务可能需要几秒钟。您可能想知道如果其中一个消费者开始一项长期任务并且只完成了部分任务而死去会发生什么。使用我们当前的代码，一旦 RabbitMQ 将消息传递给消费者，它会立即将其标记为删除。在这种情况下，如果你杀死一个工人，我们将丢失它刚刚处理的消息。我们还将丢失所有发送给该特定工作人员但尚未处理的消息。

但是我们不想丢失任何任务。如果一个工人死亡，我们希望将任务交付给另一个工人。

为了确保消息永远不会丢失，RabbitMQ 支持 消息确认。消费者发回一个确认，告诉 RabbitMQ 一个特定的消息已经被接收、处理并且 RabbitMQ 可以自由地删除它。

如果消费者在没有发送 ack 的情况下死亡（其通道关闭、连接关闭或 TCP 连接丢失），RabbitMQ 将理解消息未完全处理并将重新排队。如果同时有其他消费者在线，它会迅速将其重新发送给另一个消费者。这样，即使工人偶尔死亡，您也可以确保不会丢失任何消息。

对消费者交付确认强制执行超时（默认为 30 分钟）。这有助于检测从不确认交付的错误（卡住）消费者。可以按照`Delivery Acknowledgement Timeout`中所述增加此 超时。

默认情况下，手动消息确认是打开的。在前面的示例中，我们通过autoAck=true 标志明确地关闭了它们。一旦我们完成了一项任务，是时候将此标志设置为false并从工作人员那里发送适当的确认。

```java
channel.basicQos(1); // 一次只接受一条未确认的消息

DeliverCallback deliverCallback = (consumerTag, delivery) -> {
  String message = new String(delivery.getBody(), "UTF-8");

  System.out.println(" [x] Received '" + message + "'");
  try {
    doWork(message);
  } finally {
    System.out.println(" [x] Done");
    channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
  }
};
boolean autoAck = false;
channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, consumerTag -> { });
```

确认必须在接收交付的同一通道上发送。尝试使用不同的通道进行确认将导致通道级协议异常。

#### 消息持久性

```java
boolean durable = true;
// 持久化队列
channel.queueDeclare("hello", durable, false, false, null);
```

```java
import com.rabbitmq.client.MessageProperties;
// 持久化消息：MessageProperties.PERSISTENT_TEXT_PLAIN
channel.basicPublish("", "task_queue",
            MessageProperties.PERSISTENT_TEXT_PLAIN,
            message.getBytes());
```

**关于消息持久性的注意事项**
将消息标记为持久性并不能完全保证消息不会丢失。虽然它告诉 RabbitMQ 将消息保存到磁盘，但是当 RabbitMQ 接受消息并且还没有保存它时，仍然有很短的时间窗口。此外，RabbitMQ 不会对每条消息都执行`fsync(2)` ——它可能只是保存到缓存中而不是真正写入磁盘。持久性保证并不强，但对于我们简单的任务队列来说已经绰绰有余了。如果您需要更强的保证，那么您可以使用 发布者确认。

#### 公平调度

我们可以使用带有`prefetchCount = 1`设置的`basicQos`方法 。这告诉 RabbitMQ 一次不要给一个 worker 多条消息。或者，换句话说，在工作人员处理并确认之前的消息之前，不要向工作人员发送新消息。相反，它将把它分派给下一个不忙的工人。
![images](/images/rabbitmq-base/prefetch-count.png)

```java
int prefetchCount = 1 ; 
channel.basicQos(prefetchCount);
```

**NewTask**

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

public class NewTask {

    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);

            String message = String.join(" ", argv);

            channel.basicPublish("", TASK_QUEUE_NAME,
                    MessageProperties.PERSISTENT_TEXT_PLAIN,
                    message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + message + "'");
        }
    }

}
```

**Worker**

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class Worker {

    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        channel.basicQos(1);

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");

            System.out.println(" [x] Received '" + message + "'");
            try {
                doWork(message);
            } finally {
                System.out.println(" [x] Done");
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };
        channel.basicConsume(TASK_QUEUE_NAME, false, deliverCallback, consumerTag -> { });
    }

    private static void doWork(String task) {
        for (char ch : task.toCharArray()) {
            if (ch == '.') {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException _ignored) {
                    Thread.currentThread().interrupt();
                }
            }
        }
    }
}
```

### 发布/订阅模式（一次向多个消费者发送消息）

本质上，发布的日志消息将被广播给所有的接收者。

#### Exchanges

RabbitMQ 中消息传递模型的核心思想是生产者从不直接向队列发送任何消息。实际上，生产者通常根本不知道消息是否会被传递到任何队列。

相反，生产者只能向交换器发送消息。交换是一件非常简单的事情。一方面它接收来自生产者的消息，另一方面它将它们推送到队列中。交换必须确切地知道如何处理它收到的消息。是否应该将其附加到特定队列？它应该附加到许多队列中吗？或者它应该被丢弃。其规则由 交换类型定义。
![images](/images/rabbitmq-base/exchanges.png)

有几种可用的交换类型：`direct`、`topic`、`headers` 和`fanout`。我们将关注最后一个——`fanout`。让我们创建一个这种类型的交换，并称之为logs：

```java
channel.exchangeDeclare("logs", "fanout");
channel.basicPublish( "logs" , "" , null , message.getBytes());
```

fanout交换非常简单。它只是将收到的所有消息广播到它知道的所有队列。

#### 临时队列

当前流动的消息感兴趣，而不是对旧消息感兴趣时，可以创建一个临时队列。

- 首先，每当连接到 Rabbit 时，需要一个新的空队列。为此，可以创建一个具有随机名称的队列，或者甚至更好 - 让服务器创建一个随机队列名称。
- 其次，一旦我们断开消费者的连接，队列应该会被自动删除。

```java
String queueName = channel.queueDeclare().getQueue();
```

#### 绑定

![images](/images/rabbitmq-base/bindings.png)
告诉交换机向指定的队列发送消息。交换和队列之间的这种关系称为绑定。

```java
channel.queueBind(queueName, "logs" , "" );
```

![images](/images/rabbitmq-base/python-three-overall.png)
发送到交换机时需要在发送时提供一个`routingKey`，但它的值在`fanout`交换时会被忽略。**如果没有队列绑定到交换器，消息将丢失**

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class EmitLog {

    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

            String message = argv.length < 1 ? "info: Hello World!" :
                    String.join(" ", argv);

            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + message + "'");
        }
    }

}


import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class ReceiveLogs {
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        // 创建交换机
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        // 创建临时队列
        String queueName = channel.queueDeclare().getQueue();
        // 绑定交换机和队列
        channel.queueBind(queueName, EXCHANGE_NAME, "");

        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" + message + "'");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
    }
}
```

### 路由模式（有选择地接收消息）

仅订阅消息的子集

#### 基本绑定

发布/订阅模式中

```java
channel.queueBind(queueName, EXCHANGE_NAME, "" );
```

这可以简单地理解为：队列对来自此交换的消息感兴趣。

绑定可以采用额外的`routingKey`参数。为了避免与`basic_publish`参数混淆，我们将其称为`绑定键`。绑定键示例

```java
channel.queueBind(queueName, EXCHANGE_NAME, "black" );
```

`绑定键`的含义取决于交换类型。我们之前使用的`fanout`交换只是忽略了它的作用。

#### 直接交换

`direct`交换：路由算法很简单——消息只会进入`绑定键`和`路由键`完全匹配的队列
![images](/images/rabbitmq-base/direct-exchange.png)
在这个设置中，`直接`交换X绑定了两个队列。第一个队列使用绑定键`orange`进行绑定，第二个队列有两个绑定，一个使用绑定键`black`，另一个使用`green`。

在这样的设置中，使用路由键`orange`的消息将被路由到队列Q1。带有`black`或`green`路由键的消息将发送到Q2。所有其他消息将被丢弃。

##### 多绑定

![images](/images/rabbitmq-base/direct-exchange-multiple.png)
使用相同的绑定键绑定多个队列是完全合法的。在这种情况下，直接交换的行为类似于`fanout`并将消息广播到所有匹配的队列。带有路由键`black`的消息将被传送到`Q1`和`Q2`

##### 混合

![images](/images/rabbitmq-base/python-four.png)

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class EmitLogDirect {

  private static final String EXCHANGE_NAME = "direct_logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    try (Connection connection = factory.newConnection();
         Channel channel = connection.createChannel()) {
        channel.exchangeDeclare(EXCHANGE_NAME, "direct");

        String severity = getSeverity(argv);
        String message = getMessage(argv);

        channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes("UTF-8"));
        System.out.println(" [x] Sent '" + severity + "':'" + message + "'");
    }
  }
  //..
}


import com.rabbitmq.client.*;

public class ReceiveLogsDirect {

  private static final String EXCHANGE_NAME = "direct_logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, "direct");
    String queueName = channel.queueDeclare().getQueue();

    if (argv.length < 1) {
        System.err.println("Usage: ReceiveLogsDirect [info] [warning] [error]");
        System.exit(1);
    }

    for (String severity : argv) {
        channel.queueBind(queueName, EXCHANGE_NAME, severity);
    }
    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    DeliverCallback deliverCallback = (consumerTag, delivery) -> {
        String message = new String(delivery.getBody(), "UTF-8");
        System.out.println(" [x] Received '" +
            delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
    };
    channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
  }
}
```

### 主题模式

#### 主题交流

发送到`主题`交换的消息不能有任意的 `routing_key` - 它必须是单词列表，由点分隔。这些词可以是任何东西，但通常它们指定与消息相关的一些特征。一些有效的路由键示例："stock.usd.nyse"、"nyse.vmw"、"quick.orange.rabbit"。路由键中可以有任意多的单词，最多为`255`个字节。

绑定键也必须采用相同的格式。`主题交换`背后的逻辑类似于`直接交换`--使用特定路由键发送的消息将被传递到与匹配绑定键绑定的所有队列。但是，绑定键有两个重要的特殊情况：

- `*`（星号）可以只替换一个单词。
- `#` (hash) 可以代替零个或多个单词。

![images](/images/rabbitmq-base/python-five.png)

在这个例子中消息将使用由三个单词（两个点）组成的路由键发送。路由键中的第一个词将描述速度，第二个是颜色，第三个是物种：`<speed>.<colour>.<species>`。
我们创建了三个绑定：Q1 与绑定键“ *.orange.* ”绑定，Q2 与“ *.*.rabbit ”和“ lazy.# ”绑定。
这些绑定可以概括为：
`Q1`对所有橙色动物都感兴趣。
`Q2`想听听关于兔子的一切，以及关于懒惰动物的一切。
路由键设置为`quick.orange.rabbit`的消息将被传递到两个队列。消息`lazy.orange.elephant`也将发送给他们两个。另一方面，`quick.orange.fox`只会进入`Q1`队列，而`lazy.brown.fox`只会进入`Q2`队列。`lazy.pink.rabbit`只会被传递到`Q2`队列一次，即使它匹配两个绑定。`quick.brown.fox` 不匹配任何绑定，因此将被丢弃。

如果我们违反约定并发送带有一个或四个单词的消息，例如"orange"或"quick.orange.male.rabbit"，这些消息不会匹配任何绑定并且会丢失，"lazy.orange.male.rabbit"，即使它有四个单词，也会匹配最后一个绑定，并被发送到`Q2`队列。

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class EmitLogTopic {

  private static final String EXCHANGE_NAME = "topic_logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    try (Connection connection = factory.newConnection();
         Channel channel = connection.createChannel()) {

        channel.exchangeDeclare(EXCHANGE_NAME, "topic");

        String routingKey = getRouting(argv);
        String message = getMessage(argv);

        channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes("UTF-8"));
        System.out.println(" [x] Sent '" + routingKey + "':'" + message + "'");
    }
  }
  //..
}


import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.DeliverCallback;

public class ReceiveLogsTopic {

  private static final String EXCHANGE_NAME = "topic_logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, "topic");
    String queueName = channel.queueDeclare().getQueue();

    if (argv.length < 1) {
        System.err.println("Usage: ReceiveLogsTopic [binding_key]...");
        System.exit(1);
    }

    for (String bindingKey : argv) {
        channel.queueBind(queueName, EXCHANGE_NAME, bindingKey);
    }

    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    DeliverCallback deliverCallback = (consumerTag, delivery) -> {
        String message = new String(delivery.getBody(), "UTF-8");
        System.out.println(" [x] Received '" +
            delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
    };
    channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
  }
}
```

### RPC

---

[官网](https://www.rabbitmq.com/)
[配置说明](https://www.rabbitmq.com/configure.html)

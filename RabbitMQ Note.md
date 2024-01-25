# RabbitMQ

## 1、RabbitMQ是什么

> 就是`Erlang`语言开发的AMQP（高级消息队列协议）的开源实现

## 2、RabbitMQ的应用场景

> - 高并发消息处理（Erlang语言就是专门用来处理高并发）【高吞吐 VS 高并发】
> - 异步解耦合
> - 削峰填谷

## 3、RabbitMQ的五种工作模式

### 普通模式

> Producer将消息发送至队列，一个Consumer从此队列中消费信息

```c#
public static void SendMessage()
{
    using var connection = RabbitMQHelper.GetConnection();

    // 创建信道
    using var channel = connection.CreateModel();

    /* 创建队列
     * durable--是否持久化
     * exclusive--是否排外的:该队列仅对首次生命它的连接可见，并在断开连接时自动删除
     * autoDelete--是否自动删除
     * arguments--队列的其他参数
     */
    channel.QueueDeclare(queue: RabbitMQHelper.queueName_Normal, durable: false,
        exclusive: false, autoDelete: false, arguments: null);
    // Queue一定要指定交换机的，如果未指定，则使用默认交换机

    Console.WriteLine("Normal Producer Link Success! 请输入消息，输入exit则退出");

    string msgInfo;
    do
    {
        msgInfo = Console.ReadLine()!;  // ！:覆盖报错信息
        if (msgInfo.Trim().ToLower().Equals("exit"))
        {
            Console.WriteLine("Normal Prodecer Stoped!");
            break;
        }
        // 发送信息到RabbitMQ，使用RabbitMQ中默认提供的交换机路由，默认的RoutingKey与队列名称完全一致
        channel.BasicPublish(exchange:"", routingKey: RabbitMQHelper.queueName_Normal, false, null, Encoding.UTF8.GetBytes(msgInfo));
        Console.WriteLine($"Message Published:{msgInfo}");
    } while(true);
}
```

```c#
public static void ConsumMessage()
{
    // 消费者消费的是队列中的消息

    using var connection = RabbitMQHelper.GetConnection();

    using var channel = connection.CreateModel();

    // 不定义的话， 先启动消费端会异常
    channel.QueueDeclare(queue: RabbitMQHelper.queueName_Normal, durable: false, 
        exclusive: false, autoDelete: false, arguments: null);
    var consumer = new EventingBasicConsumer(channel);

    Console.WriteLine("Normal Consumer Link Success!");
    consumer.Received += (model, ea) =>
    {
        var message = Encoding.UTF8.GetString(ea.Body.ToArray());
        Console.WriteLine($"Message Consumed:{message}");
    };

    channel.BasicConsume(queue: RabbitMQHelper.queueName_Normal, consumer: consumer);

    Console.ReadKey();
}
```

### 工作模式

>Producer将消息发送至队列，多个Consumer从此队列中消费信息，一般情况消费过程是**轮询**的，但是在消费者端可以通过设置实现分能力消费
>
>```c#
>/* 按能力分配消息
> *   设置prefetchCount:1来告知RabbitMQ，在未收到消费端的消息确认时，不再分发消息，
> * 也就确保了当消费端处于忙碌状态时，不再分配任务
> */
>channel.BasicQos(prefetchCount: 1, prefetchSize: 0, global: false);
>```

```c#
public static void SendMessage()
{
    using var connection = RabbitMQHelper.GetConnection();

    using var channel = connection.CreateModel();

    channel.QueueDeclare(queue: RabbitMQHelper.queueName_Worker, durable: false, exclusive: false, autoDelete: false, arguments: null);

    Console.WriteLine("Worker Producer Link Success! 请输入消息，输入exit则退出");

    string msgInfo;
    do
    {
        msgInfo = Console.ReadLine()!;  // ！:覆盖报错信息
        if (msgInfo.Trim().ToLower().Equals("exit"))
        {
            Console.WriteLine("Worker Prodecer Stoped!");
            break;
        }
        // 发送信息到RabbitMQ，使用RabbitMQ中默认提供的交换机路由，默认的RoutingKey与队列名称完全一致
        channel.BasicPublish(exchange: "", routingKey: RabbitMQHelper.queueName_Worker, false, null, Encoding.UTF8.GetBytes(msgInfo));
        Console.WriteLine($"Message Published:{msgInfo}");
    } while (true);
}
```

```c#
public static void ConsumMessage()
{
    // 消费者消费的是队列中的消息

    using var connection = RabbitMQHelper.GetConnection();

    using var channel = connection.CreateModel();

    // 不定义的话， 先启动消费端会异常
    channel.QueueDeclare(queue: RabbitMQHelper.queueName_Worker, durable: false,
        exclusive: false, autoDelete: false, arguments: null);
    var consumer = new EventingBasicConsumer(channel);

    /* 按能力分配消息
     *   设置prefetchCount:1来告知RabbitMQ，在未收到消费端的消息确认时，不再分发消息，
     * 也就确保了当消费端处于忙碌状态时，不再分配任务
     */
    channel.BasicQos(prefetchCount: 1, prefetchSize: 0, global: false);

    Console.WriteLine($"Worker Consumer Link Success!");
    consumer.Received += (model, ea) =>
    {
        var message = Encoding.UTF8.GetString(ea.Body.ToArray());
        Console.WriteLine($"Message Consumed:{message}");
    };

    channel.BasicConsume(queue: RabbitMQHelper.queueName_Worker, autoAck: true, consumer: consumer);

    Console.ReadKey();
}
```

### 广播模式（发布订阅模式）

>Producer将信息发送至`Fanout`交换机中，然后交换机将信息转发至所有与其绑定的队列中（交换机不存储数据，只是实现数据的分发，数据存储在队列中）

```c#
public static void SendMessage()
{
    using var connection = RabbitMQHelper.GetConnection();
    using var channel = connection.CreateModel();

    // 声明交换机对象
    channel.ExchangeDeclare(exchange: RabbitMQHelper.exchangeName_Fanout, type: ExchangeType.Fanout);
    // 创建队列并绑定至交换机
    for (int i = 0; i < 3; ++i)
    {
        string preQueueName = RabbitMQHelper.queueNamePrefix_Fanout + i.ToString();
        channel.QueueDeclare(preQueueName, false, false, false, null);
        channel.QueueBind(queue: preQueueName, exchange: RabbitMQHelper.exchangeName_Fanout, routingKey: "");
    }

    Console.WriteLine("Fanout_Exchange Producer Link Success! 请输入消息，输入exit则退出");

    string msgInfo;
    do
    {
        msgInfo = Console.ReadLine()!;  // ！:覆盖报错信息
        if (msgInfo.Trim().ToLower().Equals("exit"))
        {
            Console.WriteLine("Fanout_Exchange Prodecer Stoped!");
            break;
        }
        channel.BasicPublish(exchange: RabbitMQHelper.exchangeName_Fanout, routingKey: "", false, null, Encoding.UTF8.GetBytes(msgInfo));
        Console.WriteLine($"Message Published:{msgInfo}");
    } while (true);
}
```

```c#
public static void ConsumMessage()
{
    using var connection = RabbitMQHelper.GetConnection();
    using var channel = connection.CreateModel();

    // 声明Exchange
    channel.ExchangeDeclare(exchange: RabbitMQHelper.queueNamePrefix_Fanout, type: ExchangeType.Fanout);
    // 创建队列并绑定至交换机
    for (int i = 0; i < 3; ++i)
    {
        string preQueueName = RabbitMQHelper.queueNamePrefix_Fanout + i.ToString();
        channel.QueueDeclare(preQueueName, false, false, false, null);
        channel.QueueBind(queue: preQueueName, exchange: RabbitMQHelper.exchangeName_Fanout, routingKey: "");
    }

    // 随机绑定某个队列
    int ind = new Random().Next(0, 3);
    Console.WriteLine($"Fanout_Exchange Consumer {ind} Link Success!");

    var consumer = new EventingBasicConsumer(channel);
    // 绑定消息接收后的事件委托
    consumer.Received += (model, ea) =>
    {
        var message = Encoding.UTF8.GetString(ea.Body.ToArray());
        Console.WriteLine($"Message Consumed:{message}");
    };

    channel.BasicConsume(queue: RabbitMQHelper.queueNamePrefix_Fanout + ind.ToString(), autoAck: true, consumer: consumer);

    Console.ReadKey();
}
```

### 路由模式（完全匹配）

>Producer将信息发送（需要指定`RoutingKey`）至`Direct`交换机中，然后交换机会根据`RoutingKe`y将消息分发至指定的队列中，一个队列可以绑定多个key

```c#
static string[] str = { "red", "yellow", "blue" };
public static void SendMessage()
{
    using var connection = RabbitMQHelper.GetConnection();
    using var channel = connection.CreateModel();

    channel.ExchangeDeclare(exchange: RabbitMQHelper.exchangeName_Direct, type: ExchangeType.Direct);

    // 创建队列并将其绑定至交换机
    for (int i = 0; i < str.Length; ++i)
    {
        string preQueueName = RabbitMQHelper.queueNamePrefix_Direct + str[i];
        channel.QueueDeclare(preQueueName, false, false, false, null);
        channel.QueueBind(exchange: RabbitMQHelper.exchangeName_Direct, queue: preQueueName, routingKey: str[i]);
    }

    Console.WriteLine("Direct_Exchange Producer Link Success! 请输入消息，输入exit则退出");

    string msgInfo;
    do
    {
        msgInfo = Console.ReadLine()!;  // ！:覆盖报错信息
        if (msgInfo.Trim().ToLower().Equals("exit"))
        {
            Console.WriteLine("Direct_Exchange Prodecer Stoped!");
            break;
        }
        // 随机在三个队列中转发
        string preRoutingKey = str[new Random().Next(0, str.Length)];
        channel.BasicPublish(exchange: RabbitMQHelper.exchangeName_Direct, routingKey: preRoutingKey, false, null, Encoding.UTF8.GetBytes(msgInfo));
        Console.WriteLine($"Message Published(To {preRoutingKey}):{msgInfo}");
    } while (true);
}
```

```c#
static string[] str = { "red", "yellow", "blue" };
public static void ConsumMessage()
{
    using var connection = RabbitMQHelper.GetConnection();
    using var channel = connection.CreateModel();

    channel.ExchangeDeclare(exchange: RabbitMQHelper.exchangeName_Direct, type: ExchangeType.Direct);

    // 创建队列并将其绑定至交换机
    for (int i = 0; i < str.Length; ++i)
    {
        string preQueueName = RabbitMQHelper.queueNamePrefix_Direct + str[i];
        channel.QueueDeclare(preQueueName, false, false, false, null);
        channel.QueueBind(exchange: RabbitMQHelper.exchangeName_Direct, queue: preQueueName, routingKey: str[i]);
    }

    // 随机绑定某个队列
    string preRoutingKey = str[new Random().Next(0, str.Length)];
    Console.WriteLine($"Direct_Exchange Consumer (RoutingKey={preRoutingKey}) Link Success!");

    var consumer = new EventingBasicConsumer(channel);
    // 绑定消息接收后的事件委托
    consumer.Received += (model, ea) =>
    {
        var message = Encoding.UTF8.GetString(ea.Body.ToArray());
        Console.WriteLine($"Message Consumed:{message}");

        // 消费完成后需要手动签收消息，否则容易导致重复消费问题
        channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: true); // 批量签收:可以降低每次签收的性能损耗
    };
    /* 消息签收模式
     * -- 手动签收:保证正确消费，不会丢消息(基于客户端而言)
     * -- 自动签收:容易丢消息
     * -- 签收:意味着消息从队列中删除
     */
    channel.BasicConsume(queue: RabbitMQHelper.queueNamePrefix_Direct + preRoutingKey, autoAck: false, consumer: consumer);

    Console.ReadKey();
}
```

### 主题模式

> Producer将消息（指定`RoutingKey`）发送至`Topic`交换机中，交换机根据`RoutingKey`将消息分发至符合要求的队列中，与路由模式的**完全匹配**不同，此模式采用的时**模糊匹配**，可以类比正则表达式

```c#
static string[] str = { "user.data.insert", "user.data.select", "user.data.update", "user.data.delete" };
static string[] rk = { "user.data.*", "user.#", "*", "#" };
public static void SendMessage()
{
    using var connection = RabbitMQHelper.GetConnection();
    using var channel = connection.CreateModel();

    // 声明交换机
    channel.ExchangeDeclare(exchange: RabbitMQHelper.exchangeName_Topic, type: ExchangeType.Topic);

    for(int i=0;i<str.Length; i++)
    {
        string preQueueName = RabbitMQHelper.queueNamePrefix_Topic + i.ToString();
        channel.QueueDeclare(preQueueName, false, false, false, null);
        channel.QueueBind(exchange: RabbitMQHelper.exchangeName_Topic, queue: preQueueName, routingKey: rk[i]);
    }

    Console.WriteLine("Topic_Exchange Producer Link Success! 请输入消息，输入exit则退出"); 

    string msgInfo;
    do
    {
        msgInfo = Console.ReadLine()!;  // ！:覆盖报错信息
        if (msgInfo.Trim().ToLower().Equals("exit"))
        {
            Console.WriteLine("Topic_Exchange Prodecer Stoped!");
            break;
        }
        string preRoutingKey = str[new Random().Next(str.Length)];
        channel.BasicPublish(exchange: RabbitMQHelper.exchangeName_Topic, routingKey: preRoutingKey, false, null, Encoding.UTF8.GetBytes(msgInfo));
        Console.WriteLine($"Message Published(To {preRoutingKey}):{msgInfo}");
    } while (true);
}
```

```c#
public static void ConsumMessage()
{
    var connection = RabbitMQHelper.GetConnection();
    var channel = connection.CreateModel();

    channel.ExchangeDeclare(exchange: RabbitMQHelper.exchangeName_Topic, type: ExchangeType.Topic);

    string queueName = RabbitMQHelper.queueNamePrefix_Topic + "1";
    channel.QueueDeclare(queueName, false, false, false, null);
    string rk = "user.data.inert";
    channel.QueueBind(exchange: RabbitMQHelper.exchangeName_Topic, queue: queueName, routingKey: rk);// 此处的routingKey无意义

    Console.WriteLine($"Topic_Exchange Consumer (RoutingKey={rk}) Link Success!");

    var consumer = new EventingBasicConsumer(channel);
    consumer.Received += (model, ea) =>
    {
        var message = Encoding.UTF8.GetString(ea.Body.ToArray());
        var rouk = ea.RoutingKey;
        Console.WriteLine($"Message Consumed:{message}:{rouk}");
    };

    channel.BasicConsume(queueName, true, consumer);

    Console.ReadKey();
}
```

### $\divideontimes$死信队列

> 当消息成为Dead Message后，可以被重新发送到死信队列，实质上还是一个交换机

> Dead Message:
>
> - 消息被拒(`basic.reject`or`basic.nack`)并且没有重新入队(`requeue=false`)
> - 当前队列中的消息数量已经超过最大长度（创建队列时`x-max-length`参数设置）
> - 消息在队列中的存活时间已经超过了预先设置的TTL时间，即过期

```c#
public static void DeadMessageDemo()
{
    using var connection = RabbitMQHelper.GetConnection();
    using var channel = connection.CreateModel();

    // 声明死信交换机
    channel.ExchangeDeclare(exchange: RabbitMQHelper.exchangeName_DLX, type: ExchangeType.Direct);
    // 声明死信队列
    channel.QueueDeclare(RabbitMQHelper.queueName_DLX, false, false, false, null);
    // 死信队列绑定死信交换机
    channel.QueueBind(exchange: RabbitMQHelper.exchangeName_DLX, queue: RabbitMQHelper.queueName_DLX, routingKey: RabbitMQHelper.queueName_DLX);

    // 声明消息交换机
    channel.ExchangeDeclare(exchange: RabbitMQHelper.exchangeName_Direct, type: ExchangeType.Direct);
    // 声明消息队列
    channel.QueueDeclare(RabbitMQHelper.queueName_Normal, false, false, false, arguments:
        new Dictionary<string, object>
        {
            /* 当一个消息队列有死信时，该队列就相当于一个生产者，把死信发送到死信队列*/
            {"x-dead-letter-exchange", RabbitMQHelper.exchangeName_DLX },   // 设置当前队列的DLX
            {"x-dead-letter-routing-key", RabbitMQHelper.queueName_DLX }    // 设置DLX的路由key，DLX会根据该值去找到消息存放的队列
        });

    // 消息队列绑定消息交换机
    channel.QueueBind(queue: RabbitMQHelper.queueName_Normal, exchange: RabbitMQHelper.exchangeName_Direct, routingKey: RabbitMQHelper.queueName_Normal);

    Console.WriteLine("DLX_QUEUE Link Success!请输入信息，输入exit则退出");

    string msgInfo;
    do
    {
        msgInfo = Console.ReadLine()!;  // ！:覆盖报错信息
        if (msgInfo.Trim().ToLower().Equals("exit"))
        {
            Console.WriteLine("Prodecer Stoped!");
            break;
        }
        channel.BasicPublish(exchange: RabbitMQHelper.exchangeName_Direct, routingKey: RabbitMQHelper.queueName_Normal, false, null, Encoding.UTF8.GetBytes(msgInfo));
        Console.WriteLine($"Message Published:{msgInfo}");
    } while (true);
}
```

```c#
public static void ConsumMessage()
{
    /*同生产者*/
  
    var consumer = new EventingBasicConsumer(channel);
    Console.WriteLine("DLX_Consumer Link Success");

    consumer.Received += (model, ea) =>
    {
        var message = Encoding.UTF8.GetString(ea.Body.ToArray());
        Console.WriteLine($"队列{RabbitMQHelper.queueName_Normal}消费消息:{message},不做ACK确认");
        channel.BasicNack(ea.DeliveryTag, multiple: false, requeue: false);
    };
    channel.BasicConsume(queue:RabbitMQHelper.queueName_Normal, autoAck:false,consumer:consumer);

    Console.ReadKey();
}
```

### $\divideontimes$延时队列

> 将普通队列添加过时时间`TTL`就成了延时队列，一般配合死信队列一起使用

```c#
channel.QueueDeclare(RabbitMQHelper.queueName_Normal, false, false, false, arguments:
    new Dictionary<string, object>
    {
        /* 当一个消息队列有死信时，该队列就相当于一个生产者，把死信发送到死信队列*/
        {"x-dead-letter-exchange", RabbitMQHelper.exchangeName_DLX },   // 设置当前队列的DLX
        {"x-dead-letter-routing-key", RabbitMQHelper.queueName_DLX },    // 设置DLX的路由key，DLX会根据该值去找到消息存放的队列
        {"x-message-ttl", 5000 }    // 设置队列的消息过期时间
    });
```


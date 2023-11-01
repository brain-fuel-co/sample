# RMQ COE

## Problems seen

On Thursday, October 26, policy changes were made as an emergency security update went through. The expectation was that those using best practices associated with queue mirroring or quorum queues would be unaffected. Those affected did not have mirrored queues.

## Solution

### Mirrored queues to replicate amongst nodes

In order to achieve a suitable HA setup, our deployed RabbitMQ configuration is set up as three-node clusters. Unfortunately, if queues are not automatically replicated from node to node, the benefits of HA are not realized, especially in cases such as performing security patches where slight blips in service might occur rolling from node to node. In older versions of RabbitMQ, the solution was to implement a policy, {"ha-mode": "all"} and apply it to all queues.

Mirrored queues were the initial solution provided by RabbitMQ for achieving high availability and data safety through queue replication. However, mirrored queues have certain limitations, particularly in terms of performance and consistency during network partitions and failovers. They rely on the master-slave replication model, which can lead to a longer data synchronization time and possible message loss during certain failure scenarios.

### The shift to quorum queues

In the evolution of RabbitMQ, one significant change regarding queue replication is the introduction and preference for quorum queues over the older mirrored queue model. This shift is evident in the RabbitMQ's ongoing updates and the messaging community's best practices.

Quorum queues, introduced in RabbitMQ 3.8, are designed to overcome these limitations. They are built on the Raft consensus algorithm, providing a more robust and predictable replication mechanism. Quorum queues ensure that the data is consistently replicated across a majority of nodes before a message is considered published, which significantly improves data safety. This replication strategy is not only more fault-tolerant but also provides better performance under various failure conditions.

For more info, see [https://www.rabbitmq.com/quorum-queues.html](https://www.rabbitmq.com/quorum-queues.html)

### Examples of Creating Quorum Queues

In each of the following examples, the key part is setting the `x-queue-type` argument to `quorum` when declaring the queue.

Java
```
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import java.util.HashMap;
import java.util.Map;

public class QuorumQueueExample {

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
	// Change the following line to specify the appropriate host;
	// This code here works if you have RMQ running on your local machine
        factory.setHost("localhost");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            String queueName = "exampleQuorumQueue";
            boolean durable = true;
            boolean exclusive = false;
            boolean autoDelete = false;
            Map<String, Object> arguments = new HashMap<>();
            arguments.put("x-queue-type", "quorum");

            channel.queueDeclare(queueName, durable, exclusive, autoDelete, arguments);
        }
    }
}

```

Spring Boot
```
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class RabbitConfig {

    @Bean
    Queue quorumQueue() {
        Map<String, Object> args = new HashMap<>();
        args.put("x-queue-type", "quorum");
        return new Queue("exampleQuorumQueue", true, false, false, args);
    }
}

```

C#
```
using RabbitMQ.Client;
using System;
using System.Collections.Generic;

class Program
{
    static void Main()
    {
	// Change the following line to specify the appropriate host;
	// This code here works if you have RMQ running on your local machine
        var factory = new ConnectionFactory() { HostName = "localhost" };
        using(var connection = factory.CreateConnection())
        using(var channel = connection.CreateModel())
        {
            var queueName = "exampleQuorumQueue";
            var durable = true;
            var exclusive = false;
            var autoDelete = false;
            var arguments = new Dictionary<string, object>
            {
                { "x-queue-type", "quorum" }
            };

            channel.QueueDeclare(queue: queueName, durable: durable, exclusive: exclusive, autoDelete: autoDelete, arguments: arguments);

            Console.WriteLine("Quorum Queue declared.");
        }
    }
}

```

### Platform Team Holiday Plans

- On Thanksgiving, 2023, at 8:00pm EST, the Platform Team will run a script ensuring the policy of {"ha-mode": "all"} is by default applied to all queues.
- If product teams make queues via the REST API or the CLI, there is no guarantee that queues will have this policy applied if something contradictory gets passed instead. Therefore, the platform team strongly recommends verifying that this policy is applied to all of your queues.
- It is incumbent upon the product teams to expediently migrate to quorum queues as traditional mirrored queues will be deprecated upon the next major RabbitMQ version update (from 3 to 4). More info to come on that later.

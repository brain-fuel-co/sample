## Dead-Letter Exchange/Queue

Here is the code for DLX/DLQ in Spring Boot, as well as listeners that send (or do not send) acknowledgments in order to take failure cases to the DLQ. Examples are given for both quorum queues and classic queues.

```
// RabbitMQConfig.java
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMQConfig {

    @Bean
    Queue regularQueue() {
        return QueueBuilder.durable("my.classic.queue")
                .withArgument("x-dead-letter-exchange", "classic.dlx")
                .build();
    }

    @Bean
    Queue classicDeadLetterQueue() {
        return QueueBuilder.durable("classic.dlq").build();
    }

    @Bean
    DirectExchange classicDeadLetterExchange() {
        return new DirectExchange("classic.dlx");
    }

    @Bean
    Binding classicDLQbinding() {
        return BindingBuilder.bind(classicDeadLetterQueue()).to(classicDeadLetterExchange()).with("");
    }

    @Bean
    Queue quorumQueue() {
        return QueueBuilder.durable("my.quorum.queue")
                .withArgument("x-dead-letter-exchange", "quorum.dlx")
                .withArgument("x-queue-type", "quorum")
                .build();
    }

    @Bean
    Queue quorumDeadLetterQueue() {
        return QueueBuilder.durable("quorum.dlq")
                .withArgument("x-queue-type", "quorum")
                .build();
    }

    @Bean
    DirectExchange quorumDeadLetterExchange() {
        return new DirectExchange("quorum.dlx");
    }

    @Bean
    Binding quorumDLQbinding() {
        return BindingBuilder.bind(quorumDeadLetterQueue()).to(quorumDeadLetterExchange()).with("");
    }
}

```

```
// MessageProcessor.java
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Service;
import com.rabbitmq.client.Channel;
import java.io.IOException;

@Service
public class MessageProcessor {

    @RabbitListener(queues = "my.classic.queue", ackMode = "MANUAL")
    public void processClassicQueueMessage(Message message, Channel channel) throws IOException {
        try {
            String messageBody = new String(message.getBody());
            System.out.println("Received from Classic Queue: " + messageBody);

            // Here would be where you would make your DB call and retry logic.

            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (Exception e) {
            System.out.println("Failed to process message from Classic Queue");
            channel.basicReject(message.getMessageProperties().getDeliveryTag(), false);
        }
    }

    @RabbitListener(queues = "my.quorum.queue", ackMode = "MANUAL")
    public void processQuorumQueueMessage(Message message, Channel channel) throws IOException {
        try {
            String messageBody = new String(message.getBody());
            System.out.println("Received from Quorum Queue: " + messageBody);

            // Here would be where you would make your DB call and retry logic

            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (Exception e) {
            System.out.println("Failed to process message from Quorum Queue");
            channel.basicReject(message.getMessageProperties().getDeliveryTag(), false);
        }
    }
}

```

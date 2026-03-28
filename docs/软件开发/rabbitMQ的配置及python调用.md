---
tags: ["python", "rabbitMQ"]
---
# rabbitMQ的配置及python调用

## 配置rabbitMQ

```bash
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3-management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: user
      RABBITMQ_DEFAULT_PASS: password
    volumes:
      - /dc/rabbitmq:/var/lib/rabbitmq
```

## 生产者

```python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost', 5672, '/', pika.PlainCredentials('user', 'password')))

channel = connection.channel()

channel.queue_declare(queue='log_queue', durable=True)
for i in range(100):
    message = "Hello World1"
    channel.basic_publish(
        exchange='',
        routing_key='log_queue',
        body=message,
        properties=pika.BasicProperties(
            delivery_mode=2,  # make message persistent
        ))
    print(" [x] Sent %r" % message)
connection.close()
```

## 消费者

```python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters('localhost', 5672, '/', pika.PlainCredentials('user', 'password')))

channel = connection.channel()

channel.queue_declare(queue='log_queue', durable=True)


def callback(ch, method, properties, body):
    print(" [x] Received %r" % body)


channel.basic_consume(queue='log_queue',
                      on_message_callback=callback,
                      auto_ack=True)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()

```

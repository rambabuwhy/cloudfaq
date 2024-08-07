# RabbitMQ

RabbitMQ is a popular message broker that enables applications to communicate with each other by sending messages through queues. Here's a step-by-step guide on how to use RabbitMQ with Golang.

#### Step 1: Install RabbitMQ

1. **Download and Install RabbitMQ:**
   * You can download RabbitMQ from here.
   * Follow the installation instructions for your operating system.
2. **Start RabbitMQ Server:**
   * On Windows, you can start RabbitMQ from the Start menu.
   *   On macOS and Linux, you can start RabbitMQ using the command:

       ```bash
       sudo rabbitmq-server
       ```

#### Step 2: Install Go and Dependencies

1. **Install Go:**
   * Download and install Go from [here](https://golang.org/dl/).
   * Follow the installation instructions for your operating system.
2. **Install the RabbitMQ Client for Go:**
   *   Open your terminal and run:

       ```bash
       go get github.com/streadway/amqp
       ```

#### Step 3: Write a Producer

Create a Go file named `producer.go` and add the following code to it:

```go
package main

import (
    "log"
    "github.com/streadway/amqp"
)

func failOnError(err error, msg string) {
    if err != nil {
        log.Fatalf("%s: %s", msg, err)
    }
}

func main() {
    conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
    failOnError(err, "Failed to connect to RabbitMQ")
    defer conn.Close()

    ch, err := conn.Channel()
    failOnError(err, "Failed to open a channel")
    defer ch.Close()

    q, err := ch.QueueDeclare(
        "hello", // name
        false,   // durable
        false,   // delete when unused
        false,   // exclusive
        false,   // no-wait
        nil,     // arguments
    )
    failOnError(err, "Failed to declare a queue")

    body := "Hello World!"
    err = ch.Publish(
        "",     // exchange
        q.Name, // routing key
        false,  // mandatory
        false,  // immediate
        amqp.Publishing {
            ContentType: "text/plain",
            Body:        []byte(body),
        })
    failOnError(err, "Failed to publish a message")
    log.Printf(" [x] Sent %s", body)
}
```

#### Step 4: Write a Consumer

Create a Go file named `consumer.go` and add the following code to it:

```go
package main

import (
    "log"
    "github.com/streadway/amqp"
)

func failOnError(err error, msg string) {
    if err != nil {
        log.Fatalf("%s: %s", msg, err)
    }
}

func main() {
    conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
    failOnError(err, "Failed to connect to RabbitMQ")
    defer conn.Close()

    ch, err := conn.Channel()
    failOnError(err, "Failed to open a channel")
    defer ch.Close()

    q, err := ch.QueueDeclare(
        "hello", // name
        false,   // durable
        false,   // delete when unused
        false,   // exclusive
        false,   // no-wait
        nil,     // arguments
    )
    failOnError(err, "Failed to declare a queue")

    msgs, err := ch.Consume(
        q.Name, // queue
        "",     // consumer
        true,   // auto-ack
        false,  // exclusive
        false,  // no-local
        false,  // no-wait
        nil,    // args
    )
    failOnError(err, "Failed to register a consumer")

    forever := make(chan bool)

    go func() {
        for d := range msgs {
            log.Printf("Received a message: %s", d.Body)
        }
    }()

    log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
    <-forever
}
```

#### Step 5: Run the Producer and Consumer

1. **Run the Consumer:**
   * Open a terminal and navigate to the directory where `consumer.go` is located.
   *   Run the following command:

       ```bash
       go run consumer.go
       ```
2. **Run the Producer:**
   * Open another terminal and navigate to the directory where `producer.go` is located.
   *   Run the following command:

       ```bash
       go run producer.go
       ```

You should see the consumer terminal output a message like:

```css
Received a message: Hello World!
```

This means your producer has successfully sent a message to the RabbitMQ queue, and your consumer has received and processed the message.

#### Step 6: Clean Up

To stop the RabbitMQ server, you can simply terminate the process or use the following command on macOS or Linux:

```bash
sudo rabbitmqctl stop
```

That's a basic setup for using RabbitMQ with Golang. You can further explore advanced features like exchanges, routing keys, and different types of queues based on your application's needs.

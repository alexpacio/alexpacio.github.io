.. title: Exchanging messages between processes (or even threads within the same program) using ZeroMQ
.. slug: message-exchanges-using-zeromq
.. date: 2025-01-27 18:00:00 UTC
.. tags: C, golang, ZeroMQ
.. category: Development
.. author: Alessandro Bolletta
.. summary: Explore how to implement efficient inter-process and inter-thread communication using ZeroMQ, a lightweight messaging library. This article demonstrates practical examples in C and Go, covering various transport protocols, thread safety considerations, Protocol Buffers integration, and performance optimization techniques. Whether you're building a multi-threaded application or need to establish communication between different processes, ZeroMQ offers a robust solution without the complexity of traditional message brokers. Learn how to implement basic messaging patterns, handle high-water marks for flow control, and serialize data effectively across different programming languages.

Inter-Process Communication with ZeroMQ (and Protocol Buffers)
==============================================================

Introduction
------------

Some may certainly say that, when you are writing so called "daemons" under Linux/Unix OSes or "services" under Windows, you might want to use OS primitives/reuse existing libraries to make your programs communicate each other. And I strongly agree with the point: it is always a good idea to use a well-tested and solid library to implement such fundamental features such as message queues.

For example, under Linux you can use D-Bus, which allows IPC at scale within the OS scope. Or, in the microservices space, you can leverage on message brokers like RabbitMQ or Kafka to stream your messages through sophisticated routing logic. However, at times you are just looking for something trivial and simple to send and queue messages where at the same time you look for brokerless setup but still you are willing to leverage on some of the features that they offer for free and with ease. That's where ZeroMQ comes in.

What is ZeroMQ?
---------------

ZeroMQ (ØMQ) is a high-performance asynchronous messaging library aimed at use in distributed or concurrent applications. It provides a message queue, but unlike message-oriented middleware, a ZeroMQ system can run without a dedicated message broker. The API layer provides a message-oriented abstraction of asynchronous network communication, multiple messaging patterns, message filtering, and more.

Supported Transport Protocols
-----------------------------

ZeroMQ supports various transport protocols through different URI schemes where the main ones are:

- ``tcp://`` - TCP transport
- ``ipc://`` - Inter-process communication (Unix domain sockets)
- ``inproc://`` - In-process communication between threads

Thread Safety and Context Management
------------------------------------

In both C and Go, ZeroMQ contexts are thread-safe, but sockets are not. Here's how to handle them properly:

In Go:

.. code-block:: go

    // Thread-safe context creation
    context, _ := zmq4.NewContext()
    
    // Socket creation should be done in the thread that uses it
    socket, _ := context.NewSocket(zmq4.PUSH)
    defer socket.Close()

In C:

.. code-block:: c

    // Thread-safe context creation
    void *context = zmq_ctx_new();
    
    // Socket creation - not thread-safe
    void *socket = zmq_socket(context, ZMQ_PUSH);

Intra-Thread Communication Example
----------------------------------

Here's an example of thread communication using ZeroMQ in C:

.. code-block:: c

    #include <zmq.h>
    #include <pthread.h>
    #include <stdio.h>
    
    void* worker_routine(void* context) {
        void* receiver = zmq_socket(context, ZMQ_PULL);
        zmq_connect(receiver, "inproc://workers");
        
        while (1) {
            char buffer[256];
            zmq_recv(receiver, buffer, 255, 0);
            printf("Received: %s\n", buffer);
        }
        
        zmq_close(receiver);
        return NULL;
    }
    
    int main() {
        void* context = zmq_ctx_new();
        void* sender = zmq_socket(context, ZMQ_PUSH);
        
        zmq_bind(sender, "inproc://workers");
        
        pthread_t worker;
        pthread_create(&worker, NULL, worker_routine, context);
        
        // Send messages
        const char* message = "Hello Worker!";
        zmq_send(sender, message, strlen(message), 0);
        
        sleep(1);  // Allow time for message processing
        
        zmq_close(sender);
        zmq_ctx_destroy(context);
        return 0;
    }

High Water Mark and Flow Control
--------------------------------

ZeroMQ provides flow control through the High Water Mark (HWM) feature. When the HWM is reached, ZeroMQ will either block or drop messages depending on the socket type and configuration:

.. code-block:: c

    int hwm = 1000;
    zmq_setsockopt(socket, ZMQ_SNDHWM, &hwm, sizeof(hwm));
    
    // To prevent dropping messages when HWM is reached
    int nodrop = 1;
    zmq_setsockopt(socket, ZMQ_XPUB_NODROP, &nodrop, sizeof(nodrop));

Protocol Buffers Integration
----------------------------

Since ZeroMQ only transfers raw bytes, it pairs well with Protocol Buffers for structured data serialization. Here's an example using both C++ and Go:

First, define your protocol buffer:

.. code-block:: protobuf

    // message.proto
    syntax = "proto3";
    
    message DataMessage {
        string content = 1;
        int64 timestamp = 2;
    }

Using it in Go:

.. code-block:: go

    package main

    import (
        "log"

        "github.com/pebbe/zmq4"
        "google.golang.org/protobuf/proto"
        examplepb "path/to/generated/proto"
    )

    func main() {
        // Create a ZeroMQ context
        context, err := zmq4.NewContext()
        if err != nil {
            log.Fatalf("Failed to create ZeroMQ context: %v", err)
        }
        defer context.Term() // Ensure the context is terminated when the program exits

        // Create a ZeroMQ Subscriber socket
        subscriber, err := context.NewSocket(zmq4.SUB)
        if err != nil {
            log.Fatalf("Failed to create subscriber socket: %v", err)
        }
        defer subscriber.Close()

        // Connect to the publisher
        err = subscriber.Connect("tcp://127.0.0.1:5555")
        if err != nil {
            log.Fatalf("Failed to connect subscriber: %v", err)
        }

        // Subscribe to all messages
        err = subscriber.SetSubscribe("")
        if err != nil {
            log.Fatalf("Failed to set subscription: %v", err)
        }

        log.Println("Subscriber started, waiting for messages...")

        for {
            // Receive the serialized message
            data, err := subscriber.RecvBytes(0)
            if err != nil {
                log.Printf("Failed to receive message: %v", err)
                continue
            }

            // Deserialize the message
            var message examplepb.ExampleMessage
            err = proto.Unmarshal(data, &message)
            if err != nil {
                log.Printf("Failed to deserialize message: %v", err)
                continue
            }

            // Print the received message
            log.Printf("Received message: ID=%s, Content=%s", message.Id, message.Content)
        }
    }


Explanation
-----------

Context: The zmq.NewContext() function creates a new ZeroMQ context, which is required to create sockets.

Socket: The context.NewSocket(zmq.SUB) function creates a new SUB socket for subscribing to messages.

Connect: The subscriber.Connect("tcp://localhost:5555") function connects the subscriber to the publisher’s address.

Subscribe: The socket.SetSubscribe("") function subscribes to all messages (an empty string means subscribe to everything). This acts as a way to subscribe to a string prefix (so called "topic" in other MQ systems)

Recv: The socket.RecvBytes(0) function blocks until a message is received.

Asynchronous Message Emission
-----------------------------

ZeroMQ supports non-blocking sends using the ``ZMQ_DONTWAIT`` flag:

.. code-block:: c

    zmq_send(socket, message, size, ZMQ_DONTWAIT);
    // Code continues immediately without waiting for the send operation outcome

Performance Considerations
--------------------------

ZeroMQ is particularly well-suited for high-performance scenarios where:

1. You need to decouple the message producer from the consumer
2. The critical section needs to emit messages without blocking
3. You want to avoid the overhead of a message broker
4. You need reliable message delivery without managing it yourself

The library handles many complex aspects automatically:

- Message framing
- Connection handling and reconnection
- Message queuing
- Fair message distribution
- Transport abstraction


ZeroMQ Messaging Patterns
-------------------------

ZeroMQ supports several fundamental messaging patterns, each designed for specific use cases:

Push/Pull (Pipeline)
~~~~~~~~~~~~~~~~~~~~

The Push/Pull pattern creates a one-way data distribution pipeline. Messages sent by pushers are load-balanced among all connected pullers.

.. code-block:: c

    // Pusher (sender)
    void *pusher = zmq_socket(context, ZMQ_PUSH);
    zmq_bind(pusher, "tcp://*:5557");
    
    // Puller (receiver)
    void *puller = zmq_socket(context, ZMQ_PULL);
    zmq_connect(puller, "tcp://localhost:5557");

Use cases:
- Parallel task distribution
- Workload distribution in producer/consumer scenarios
- Data pipeline processing

Pub/Sub (Publisher/Subscriber)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Publishers send messages while subscribers receive them based on topics. Each subscriber can subscribe to multiple topics.

.. code-block:: go

    // Publisher
    publisher, _ := zmq4.NewSocket(zmq4.PUB)
    publisher.Bind("tcp://*:5563")
    
    // Send message with topic
    publisher.Send("weather.london temperature:22", 0)
    
    // Subscriber
    subscriber, _ := zmq4.NewSocket(zmq4.SUB)
    subscriber.Connect("tcp://localhost:5563")
    subscriber.SetSubscribe("weather.london")

Use cases:
- Event broadcasting
- Real-time data feeds
- System monitoring
- Live updates

Request/Reply (REQ/REP)
~~~~~~~~~~~~~~~~~~~~~~~

A synchronous pattern where each request must be followed by a reply.

.. code-block:: c

    // Server (Reply)
    void *responder = zmq_socket(context, ZMQ_REP);
    zmq_bind(responder, "tcp://*:5555");
    
    // Client (Request)
    void *requester = zmq_socket(context, ZMQ_REQ);
    zmq_connect(requester, "tcp://localhost:5555");

Use cases:
- Remote procedure calls (RPC)
- Service APIs
- Task delegation with acknowledgment

Dealer/Router
~~~~~~~~~~~~~

An advanced asynchronous pattern that allows for complex routing scenarios.

.. code-block:: go

    // Router
    router, _ := zmq4.NewSocket(zmq4.ROUTER)
    router.Bind("tcp://*:5555")
    
    // Dealer
    dealer, _ := zmq4.NewSocket(zmq4.DEALER)
    dealer.Connect("tcp://localhost:5555")

Use cases:
- Load balancing
- Asynchronous request/reply
- Complex routing topologies
- Service meshes

Pattern Selection Guidelines
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When choosing a pattern, consider:

1. Message Flow Direction
   - One-way: Push/Pull or Pub/Sub
   - Two-way: Request/Reply or Dealer/Router

2. Synchronization Requirements
   - Synchronous: Request/Reply
   - Asynchronous: Push/Pull, Pub/Sub, Dealer/Router

3. Scalability Needs
   - Fan-out: Pub/Sub
   - Load balancing: Push/Pull or Dealer/Router
   - Both: Combination of patterns

4. Message Delivery Guarantees
   - At-most-once: Pub/Sub
   - At-least-once: Request/Reply
   - Custom guarantees: Dealer/Router

Example: Combining Patterns
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here's an example combining Pub/Sub with Push/Pull for a logging system:

.. code-block:: go

    package main
    
    import (
        "github.com/pebbe/zmq4"
        "log"
    )
    
    func main() {
        // Event publisher
        publisher, _ := zmq4.NewSocket(zmq4.PUB)
        publisher.Bind("tcp://*:5563")
        
        // Log collector
        collector, _ := zmq4.NewSocket(zmq4.PULL)
        collector.Bind("tcp://*:5564")
        
        // Worker that processes logs and publishes events
        go func() {
            worker, _ := zmq4.NewSocket(zmq4.PUSH)
            worker.Connect("tcp://localhost:5564")
            
            subscriber, _ := zmq4.NewSocket(zmq4.SUB)
            subscriber.Connect("tcp://localhost:5563")
            subscriber.SetSubscribe("error")
            
            // Process messages...
        }()
    }

This setup allows for:
- Real-time error broadcasting (Pub/Sub)
- Reliable log collection (Push/Pull)
- Scalable processing workers
- Decoupled components

The choice of pattern significantly impacts your system's behavior, performance, and scalability. It's often beneficial to combine patterns to achieve more complex messaging requirements while maintaining simplicity in individual components.

Conclusion
----------

When you need a lightweight, broker-less messaging solution with good performance characteristics, ZeroMQ provides an excellent balance of features and simplicity. It's particularly valuable in scenarios where you need to quickly implement reliable inter-process or inter-thread communication without the overhead of a full message broker infrastructure.

While it may not replace more robust solutions like Kafka for large-scale distributed systems, ZeroMQ fills an important niche for local and small-scale distributed messaging needs, especially when performance is a critical factor.

Useful Links
------------

- ZeroMQ Official Website: https://zeromq.org/
- ZeroMQ Documentation: https://zeromq.org/get-started/
- Protocol Buffers Official Website: https://protobuf.dev/
- Protocol Buffers Documentation: https://protobuf.dev/overview/
- ZeroMQ Guide: https://zguide.zeromq.org/
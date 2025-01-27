.. title: Exchanging messages between processes (or even threads within the same program) using ZeroMQ
.. slug: message-exchanges-using-zeromq
.. date: 2025-01-27 20:00:00 UTC
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

ZeroMQ supports various transport protocols through different URI schemes:

- ``tcp://`` - TCP transport
- ``ipc://`` - Inter-process communication (Unix domain sockets)
- ``inproc://`` - In-process communication between threads
- ``pgm://`` - Pragmatic General Multicast
- ``epgm://`` - Encapsulated PGM

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
        pb "path/to/generated/proto"
        "github.com/pebbe/zmq4"
        "google.golang.org/protobuf/proto"
    )
    
    func main() {
        socket, _ := zmq4.NewSocket(zmq4.PULL)
        defer socket.Close()
        
        socket.Bind("tcp://*:5555")
        
        for {
            data, _ := socket.RecvBytes(0)
            msg := &pb.DataMessage{}
            proto.Unmarshal(data, msg)
            log.Printf("Received: %s at %d", msg.Content, msg.Timestamp)
        }
    }

Asynchronous Message Emission
-----------------------------

ZeroMQ supports non-blocking sends using the ``ZMQ_DONTWAIT`` flag:

.. code-block:: c

    zmq_send(socket, message, size, ZMQ_DONTWAIT);
    // Code continues immediately without waiting

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
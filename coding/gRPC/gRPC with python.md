# gRPC with Python

Welcome to gRPC! Think of gRPC as a high-speed, modern cousin to REST APIs. Instead of sending bulky human-readable JSON text over HTTP/1.1, gRPC serializes your data into compact binary bytes using **Protocol Buffers** and runs over **HTTP/2**. This makes it blazing fast, strongly typed, and capable of bidirectional streaming—perfect for microservices.


---

###  What is gRPC? (In a Nutshell)

* **Contract-First:** You define your data structures and service APIs in a neutral `.proto` file first.
* **Code Generation:** Tools automatically generate client and server boilerplate code in Python (or dozens of other languages).
* **Binary Serialization (Protobuf):** Data is packed into tiny binary payloads, saving bandwidth and CPU cycles compared to JSON.
* **HTTP/2 Powered:** Supports multiplexing (sending multiple requests over a single TCP connection) and streaming.

---

## How to build a gRPC service

Here is your step-by-step guide and note for building a gRPC service in Python.

### 1. Define the Service in a `.proto` File

Create a file named `greeter.proto`. This file acts as the contract between your client and server.

```proto
syntax = "proto3";

package greeter;

// The Greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings.
message HelloReply {
  string message = 1;
}

```

---

### 2. Generate Python Stubs (`pb2` and `pb2_grpc`)

First, make sure you have installed the required gRPC packages in your Python environment:

```bash
pip install grpcio grpcio-tools

```

To compile your `.proto` file into Python modules (`greeter_pb2.py` and `greeter_pb2_grpc.py`), run the following command in your terminal:

```bash
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. greeter.proto

```

* **`-I.`**: Tells the compiler to look in the current directory for proto files.
* **`--python_out=.`**: Generates `greeter_pb2.py`, which handles data serialization and message classes.
* **`--grpc_python_out=.`**: Generates `greeter_pb2_grpc.py`, which contains the server and client classes.

---

### 3. Write the Server-Side and Client-Side Scripts

Now that your stubs are generated, you can implement the server logic and client logic.

#### A. The Server Script (`server.py`)

The server inherits from the generated `Servicer` class, implements your business logic, and starts listening on a port.

```python
from concurrent import futures
import grpc
import greeter_pb2
import greeter_pb2_grpc

# 1. Inherit from the generated Servicer class
class GreeterService(greeter_pb2_grpc.GreeterServicer):
    def SayHello(self, request, context):
        print(f"Received request from: {request.name}")
        # Return a response using the generated message class
        return greeter_pb2.HelloReply(message=f"Hello, {request.name}! Welcome to gRPC.")

def serve():
    # 2. Create a gRPC server with a ThreadPool
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    
    # 3. Register our service with the server
    greeter_pb2_grpc.add_GreeterServicer_to_server(GreeterService(), server)
    
    # 4. Bind port and start
    server.add_insecure_port('[::]:50051')
    server.start()
    print("gRPC Server is running on port 50051...")
    server.wait_for_termination()

if __name__ == '__main__':
    serve()

```

#### B. The Client Script (`client.py`)

The client opens an insecure channel to the server, creates a stub, and calls the remote method just like a local function.

```python
import grpc
import greeter_pb2
import greeter_pb2_grpc

def run():
    # 1. Connect to the server via an insecure channel
    with grpc.insecure_channel('localhost:50051') as channel:
        # 2. Create a stub (client)
        stub = greeter_pb2_grpc.GreeterStub(channel)
        
        # 3. Call the remote procedure
        request = greeter_pb2.HelloRequest(name='Alice')
        response = stub.SayHello(request)
        
        print(f"Client received response: {response.message}")

if __name__ == '__main__':
    run()

```

---

### 4. How to Test It:

1. Open a terminal and run your server:
```bash
python server.py

```


2. Open a second terminal and run your client:
```bash
python client.py

```


You should see the client print the greeting message sent back from the server!
---
title: "Grains"
date: 2020-05-28T16:34:24+02:00
draft: false
tags: [protoactor, docs]
---

# Proto.Cluster

## Virtual Actors, aka. Grains

Proto.Cluster leverages the *"Vritual Actor Model"*, which was pioneered by Microsoft Orleans.
Unlike the traditional Actor Model used in Erlang or Akka, where developers must care about actor lifecycles, placement and failures.
The virtual actor model instead focus on ease of use, high availability where most of the complexity have been abstracted away from the developer.

> The Microsoft Orleans website describes this as *A straightforward approach to building distributed, high-scale applications in .NET*.

Proto.Actor combines this way of clustering, with the traditional actor model to combine the best of both worlds.
This allows us to create huge clusters of stateful services where the virtual actors acts as entry points which in turn can contain entire graphs of local actors.

This offers us a unique way to optimize for data locality, while still offering ease of use at scale.

Just like everything else in Proto.Actor where we have re-used proven technologies such as Protobuf and gRPC, we do the same for clustering, we do not reinvent the wheel and create our own cluster mechanics.
Instead, we leverage proven technologies such as Consul, ETCD or Kubernetes to power our Cluster member management.

The short version of what virtual actors are, is that they are abstractions on top of plain actors.
They are spawned *somewhere* in your cluster, and their lifecycle is managed by the cluster instead of you.

This means that you as a developer, don't have to care or know if the actor already exists or where it exists.
You address it using its identity and kind and the cluster does the rest for you.

### Partition Identity

For more information about the details of cluster mechanics.
See [Cluster Partitions](cluster-partitions.md)

### Persistent Identity

** TODO **

## FAQ

### Communicate with Virtual Actors

Virtual actors require request-response based messaging, this is to ensure that the message was delivered and/or that the actor was properly activated.

You do this using `cluster.RequestAsync("MyActor123", "SomeKind", someMessage, cancellationToken)` in .NET, and `cluster.call("MyActor123", "SomeKind", someMessage)` using Go.

This allows Proto.Actor to re-try getting the PID by calling the actor until it succeeds.

### Generate Typed Virtual Actors

The first thing you need to do is to define your messages and grain contracts.
You do this by using Protobuf IDL files.

This is the definition from the `/examples/cluster/shared` example

```protobuf
syntax = "proto3";
package shared;

message HelloRequest {
  string name = 1;
}

message HelloResponse {
  string message = 1;
}

message AddRequest {
  double a = 1;
  double b = 2;
}

message AddResponse {
  double result = 1;
}

service Hello {
  rpc SayHello (HelloRequest) returns (HelloResponse) {} 
  rpc Add(AddRequest) returns (AddResponse) {}
}
```

Once you have this, you can generate your code using protoc.

**Windows**
```text
protoc -I=. -I=%GOPATH%\src --gogoslick_out=. protos.proto 
protoc -I=. -I=%GOPATH%\src --gorleans_out=. protos.proto 
```

## Implementing

When the contracts have been generated, you can start implementing your grains:

```go
package shared

//a Go struct implementing the Hello interface
type hello struct {
}

func (*hello) SayHello(r *HelloRequest) *HelloResponse {
	return &HelloResponse{Message: "hello " + r.Name}
}

func (*hello) Add(r *AddRequest) *AddResponse {
	return &AddResponse{Result: r.A + r.B}
}

//Register what implementation GAM should use when 
//creating actors for a certain grain type.
func init() {
	//apply DI and setup logic
	HelloFactory(func() Hello { return &hello{} })
}
```

Show examples how to codegen grains

## Hybrid Clusters: C#, Go and Kotlin

//TODO

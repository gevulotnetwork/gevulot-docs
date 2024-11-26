# gRPC

Gevulot provides all of its on-chain services through [gRPC](https://grpc.io/) API. All protobuf definitions for the services can be found from the [gevulot-rs GitHub repo](https://github.com/gevulotnetwork/gevulot-rs/tree/main/proto/gevulot/gevulot).

The service API is split by the on-chain data types:

* Pins
* Tasks
* Workflows
* Workers

The preferred way to consume the gRPC service is by using [gevulot-rs](gevulot-rs.md) library as it provides important helper functionality, such as suppoort for event watching etc.

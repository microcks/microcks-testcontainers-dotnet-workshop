# Step 3: Local development experience with Microcks

Our application uses Kafka and external dependencies.

Currently, if you run the application from your terminal, you will see the following error:

```shell
dotnet run --project src/Order.Service

Génération...
dbug: Microsoft.Extensions.Hosting.Internal.Host[1]
      Hosting starting
[...]
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5088
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /Users/laurent/Development/github/microcks-testcontainers-dotnet-workshop/src/Order.Service
dbug: Microsoft.Extensions.Hosting.Internal.Host[2]
      Hosting started
%3|1780646312.515|FAIL|order-service-consumer#consumer-1| [thrd:localhost:9092/bootstrap]: localhost:9092/bootstrap: Connect to ipv6#[::1]:9092 failed: Connection refused (after 0ms in state CONNECT)
%3|1780646312.515|ERROR|order-service-consumer#consumer-1| [thrd:app]: order-service-consumer#consumer-1: localhost:9092/bootstrap: Connect to ipv6#[::1]:9092 failed: Connection refused (after 0ms in state CONNECT)
%3|1780646312.515|ERROR|order-service-consumer#consumer-1| [thrd:localhost:9092/bootstrap]: 1/1 brokers are down
[...]
```

You can test the app by creating a simple order with this command:

```shell
curl -XPOST localhost:8080/api/orders -H 'Content-type: application/json' \
    -d '{"customerId": "lbroudoux", "productQuantities": [{"productName": "Millefeuille", "quantity": 1}], "totalPrice": 5.1}'
```

But you'll also get this error:

```shell
{"type":"https://tools.ietf.org/html/rfc9110#section-15.6.1","title":"An error occurred while processing your request.","status":500,"detail":"Connection refused (localhost:9090)","traceId":"00-ccc228108cefc9f100ada55034e686fa-46d2dcf59e727823-00"}
```

> [!Important]
> To run the application locally, we need to have a Kafka broker up and running + the other dependencies corresponding to our Pastry API provider and reviewing system.

Instead of installing these services on our local machine, or using Docker to run these services manually, we will use a utility tool with this simple command `microcks.sh`. Microcks docker-compose file (`microcks-docker-compose.yml`) has been configured to automatically import the Order API contract but also the Pastry API contracts. Both APIs are discovered on startup and Microcks UI should be available on http://localhost:9090 in your browser:

```shell
./microcks.sh
[+] Running 4/4
 ✔ Container microcks-kafka                                  Started                                              0.1s 
 ✔ Container microcks-testcontainers-dotnet-demo-microcks-1  Started                                              0.1s 
 ✔ Container microcks-async-minion                           Started                                              0.1s 
 ✔ Container microcks-testcontainers-dotnet-demo-importer-1  Started                                              0.1s 
```

> [!TIP]
> Services from the `microcks-docker-compose.yml` file use host ports: 9090 (microcks), 9091 (microcks-async), 9092 (kafka). If those ports are already used on your machine, you can easily edit the compose file and change them. You'll need to adapt future directives though.

Because our `Order Service` application has been configured to talk to Microcks mocks (see the default settings in `appsettings.Development.json` and `launchSettings.json`), you should be able to directly call the `Order API` and invoke the whole chain made of the 3 components:

And that's it! 🎉 You don't need to download and install extra-things, or clone other repositories and figure out how to start your dependant services. 

Start again your application from terminal using `dotnet run --project src/Order.Service` command and verify that the application starts successfully. 🙌

```shell
[...]
dbug: Microsoft.Extensions.Hosting.Internal.Host[1]
      Hosting starting
info: Order.Service.OrderEventConsumerHostedService[0]
      Started consuming from 'orders-reviewed' topic
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5088
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /Users/laurent/Development/github/microcks-testcontainers-dotnet-workshop/src/Order.Service
dbug: Microsoft.Extensions.Hosting.Internal.Host[2]
      Hosting started
fail: Order.Service.OrderEventConsumerHostedService[0]
      Error consuming message from Kafka
      Confluent.Kafka.ConsumeException: Subscribed topic not available: orders-reviewed: Broker: Unknown topic or partition
         at Confluent.Kafka.Consumer`2.Consume(Int32 millisecondsTimeout)
         at Confluent.Kafka.Consumer`2.Consume(CancellationToken cancellationToken)
         at Order.Service.OrderEventConsumerHostedService.ExecuteAsync(CancellationToken stoppingToken) in /Users/laurent/Development/github/microcks-testcontainers-dotnet-workshop/src/Order.Service/OrderEventConsumerHostedService.cs:line 64
[...]
```

> We have a remaining failure here but it's actually more a warning than a failure, it has no impact on the workshop follow-up.

Now, you can invoke the APIs using CURL or Postman or any of your favourite HTTP Client tools.

## Create an order

```shell
curl -POST localhost:5088/api/orders -H 'Content-type: application/json' \
    -d '{"customerId": "lbroudoux", "productQuantities": [{"productName": "Millefeuille", "quantity": 1}], "totalPrice": 5.1}' -v
```

You should get a response similar to the following:

```shell
< HTTP/1.1 201 Created
< Content-Type: application/json; charset=utf-8
< Date: Fri, 05 Jun 2026 08:15:31 GMT
< Server: Kestrel
< Location: /api/orders/2860a6da-b1ff-46a0-b8dc-b17ceea64166
< Transfer-Encoding: chunked
< 
* Connection #0 to host localhost left intact
{"id":"2860a6da-b1ff-46a0-b8dc-b17ceea64166","status":"Created","customerId":"lbroudoux","productQuantities":[{"productName":"Millefeuille","quantity":1}],"totalPrice":5.1}
```

Now test with something else, requesting for another Pastry:

```shell
curl -POST localhost:5088/api/orders -H 'Content-type: application/json' \
    -d '{"customerId": "lbroudoux", "productQuantities": [{"productName": "Eclair Chocolat", "quantity": 1}], "totalPrice": 4.1}' -v
```

This time you get another "exception" response:

```shell
< HTTP/1.1 422 Unprocessable Entity
< Content-Type: application/json; charset=utf-8
< Date: Fri, 05 Jun 2026 08:17:28 GMT
< Server: Kestrel
< Transfer-Encoding: chunked
< 
* Connection #0 to host localhost left intact
{"productName":"Eclair Chocolat","details":"Pastry 'Eclair Chocolat' is unavailable."}%
```

and this is because Microcks has created different simulations for the Pastry API 3rd party API based on API artifacts we loaded. Check the `tests/resources/third-parties/apipastries-openapi.yml` and `tests/resources/third-parties/apipastries-postman-collection.json` files to get details.

### 🎁 Bonus step - Explore Microcks UI and understand dispatchers

* Use `docker ps` command or Docker Desktop to retrieve the local port where Microcks is actually started and open it in your browser
* Review the mocked APIs:

  * How can you visually check that the **API Pastries - 0.0.1** mock is called?
  * How can you figure out that Microcks is sending back the correct response?

* Explore [Dispatcher & dispatching rules](https://microcks.io/documentation/explanations/dispatching/) to learn more.

### 
[Next](step4-write-rest-tests.md)

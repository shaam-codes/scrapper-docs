# Documentation for scrapper system
This is for complete documentation repo, which includes functional and technical related documents with images
## Architecture

This system follows microservice architecture and each service is executed independently and scales on its own. Also Each service developed in its own repos and deployments can be done individually. All the services use docker containers and the deployer will be used for deploy stage environments and productions.
A high level diagram of services as below

![high level diagram of services](.//diagrams/arhitecture.png "high level diagram of services")

## API Gateway
We use Envoy proxy as the proxy as the gateway. This manages the authentication and manages ngress for the web servers. 

## Authentication

As name refers, this manages the authentication process and updates the header with necessary details.
## Core Application

This runs on web servers. Each server has their own web servers to communicate with Envoy proxy. [Nginx Unit](https://unit.nginx.org/) used as the web server, it runs as the application server and web server.

Application builds with pure Node Js and [gPRC](https://grpc.io/docs/languages/node/) will be used to communicate from outside. Basically API gateway and worker nodes will communicate for now.

These servers should scale out and be based on the network traffic. Since these servers are behind the API gateway, no need of authentication needed to access these servers but expecting relevant arguments to access the service.

Application build using [Hexagonal Architecture](https://en.wikipedia.org/wiki/Hexagonal_architecture_(software)) to isolate business logic from the framework or any other dependencies such database or message brokers, etc. Also helps to execute tests in it without real dependencies.

![Hexagonal Architecture](.//diagrams/hexagonal.drawio.png "Hexagonal Architecture")

## scheduler worker

This works in a interval to fetch active sensor chambers and push to the message queue if they're ready to collect the data (collection intervals can be defined by user per chamber). For the time being only one instance is enough for the requirements, will decide later if needed further, if so we may consider the duplication of messages.

## worker

These are listeners or subscribers of the message queue, if any new message available in the queue mark as acknowledged and process it. Basically this will send the request to the chamber and store the response in the main storage. we may need to use any other data storage as an intermediate data storage (stream/cache) and need to define another worker to persistent data into postgres
For requesting sensor chambers uses envoy egress (TBD)

## Queue

This uses [BullMQ](https://docs.bullmq.io/) to store each event, ideally all the events or requests for the chamber stored here, then workers will execute one by one. No automated or scheduled for fetch values from the chamber. Scheduler workers can only push the events here.
## Redis

As usual this is used for reducing requests to persistent data storage and performance. Store some most frequent details such as active chambers, etc.
Also this use by some functions by API Gateway such as rate limiter and authentication

## Monitoring

As usual all logs, errors and also system or infrastructure stats monitor here. Service uses [OpenTelemetry](https://opentelemetry.io/).
## Database

PostgreSQL uses as the relational database for the system and its primary data storage. we may some other databases such elastic search or some more caches for query improvements or any other performance improvements later

here is database ERD![ERD for the database](.//diagrams/schema.png "ERD for the database")


[draw.io](https://app.diagrams.net/#Hshaam-codes%2Fscrapper-docs%2Fmain%2Fscrapper.drawio)
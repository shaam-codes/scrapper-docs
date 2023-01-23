# Documentation for scrapper system
This is for complete documetation repo, which includes functionanl and technical related documents with images

## Architecture
This is system follows micro-service arhitecture and each sevices are executing independanly and scales in own. Also Each sevice developed in own repos and deployments can be done individually. All the services uses docker containers and deployer will be used for deploy stage environments and producitons.

A high level diagram of services as below

![high level diagram of services](.//diagrams/arhitecture.png "high level diagram of services")

## API Gateway
We use Envoy proxy as the proxy as the gateway. This manages the autheticaiton and manages ngress for the web servers. 

## Authtication
As name refers, this manages the authentication process and update the header with necessary details.

## Core Application
This runs on web servers. Each server having their own web servers to communicate with Envoy proxy. [Nginx Unit](https://unit.nginx.org/) used as the web server, it runs as the application server and web server.

Application build with pure Node Js and [gPRC](https://grpc.io/docs/languages/node/) will used to communicate from out side. Basically API gateway and worker nodes will communicate for now.

These servers should scale out and in based on the network traffic. Since these servers behind the API gateway, no need of authentication needed to access these server but expecting relevant arguments to access the service.

Applivation build using [Hexagonl Architecture](https://en.wikipedia.org/wiki/Hexagonal_architecture_(software)) to isolate business logic from the framework or any other dependacies such database or message brokers, etc. Also helps to exeute tests in it without real dependancies.

## schedular worker
This works in a interval to fetch active sensor chambers and push to message queue if they're ready to collect the data (colleciton intervals can be defined by user per chamber). For the time being only one instance enough for the requirements, will decide later if needed further, if so we may consider the duplicaiton of messages.


## worker
These are listener or subscibers of the message queue, if any new message avaialbe in the queue mark as acknowledged and process it. Basically this will send the request to the chamber and store the response in the main storage. we may need to use any other data storeage as an intermediate data storage (stream/cache) and need to define another worker to persistant data into postgres

For requestting sensor chambers uses envoy egress (TBD)

## Queue
This uses [BullMQ](https://docs.bullmq.io/) to store each events, ideally all the events or request for the chamber stored here, then worker will execute one by one. No any automated or scheduale for fetch values from chamber. Schedular worker can only push the events here.

## Redis
As usual this uses for reduce request to persistant data storage and performance. Store some most frequest details such as active chambers and etc.

Also this use by some functions by API Gateway such as rate limiter and authenticaiton

## Monitoring
As usual all logs, errors and also system or infrastructure stas monitor here. Service uses [OpenTelemetry](https://opentelemetry.io/).

## Database
Postgre SQL uses as the relational database for the system and it's primary data storage. we may some other databases such elastic search or some more caches for query improvements or any other performance improvements later

here is database ERD
![ERD for the database](.//diagrams/schema.png "ERD for the database")

[draw.io](https://app.diagrams.net/#Hshaam-codes%2Fscrapper-docs%2Fmain%2Fscrapper.drawio)

﻿
#Installing rabbitMQ

sudo docker run -d --hostname local-rabbit-mq --name plural-spring-cloud-rabbitmq -p 15672:15672 -p 5672:5672 rabbitmq:3.7.25-management
sudo docker images – see all images
sudo docker ps   - see all running containers

sudo docker stop plural-spring-cloud-rabbitmq  - stop image plural-spring-cloud-rabbitmq  
sudo docker ps -all      - see all containers including stopped
sudo docker start  plural-spring-cloud-rabbitmq

http://localhost:15672 – RabbitMQ Docker console
username: guest
password: guest



#Partitions
1. mvn package


#goto to target forlder and run 3 terminals in maven target folder
2. cd /home/osboxes/eclipse-workspace/spring-cloud-coordinating-services-ch06-stream-tollintake/target

#run jar files
3.
java -jar spring-cloud-coordinating-services-ch06-stream-tollintake-0.0.1-SNAPSHOT.jar --spring.cloud.stream.instanceIndex=0

java -jar spring-cloud-coordinating-services-ch06-stream-tollintake-0.0.1-SNAPSHOT.jar --spring.cloud.stream.instanceIndex=1

java -jar spring-cloud-coordinating-services-ch06-stream-tollintake-0.0.1-SNAPSHOT.jar --spring.cloud.stream.instanceIndex=2

http://localhost:15672/#/queues
should be created
fastpasstoll.tollProcessingGroup-0
D	idle	0	0	0			
fastpasstoll.tollProcessingGroup-1
D	idle	0	0	0			
fastpasstoll.tollProcessingGroup-2

4. Send multiple POST request to

POST http://localhost:8082/toll
{"stationId":10, "customerId":100, "timestamp": "2020-04-17T12:04:0"}

POST http://localhost:8082/toll
{"stationId":20, "customerId":100, "timestamp": "2020-04-17T12:04:0"}


POST http://localhost:8082/toll
{"stationId":30, "customerId":100, "timestamp": "2020-04-17T12:04:0"}


#installing MYSQL on Docker
sudo docker pull mysql:latest

#check images
sudo docker images

#install REDIS on Docker
sudo docker pull redis:latest

#Run container on already downloaded image
#run mysql container
sudo docker run --name pluralsigth-spring-cloud-mysql -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=spring-cloud-cs-data-flow -e MYSQL_ROOT_HOST=%  -p 3306:3306 -d mysql:latest

#resolve docker mysql Access denied for user 'root'@'172.17.0.1'
select * from mysql.user;
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;

#run Redis container
sudo docker run --name pluralsight-spring-cloud-redis -p 6379:6379 -d redis:latest


#docker run bash into container
sudo docker exec -it <container_id_or_name> 

#run dataflow server and dataflow shell
cd /home/osboxes/eclipse-workspace/spring-cloud-coordinating-services-ch07-dataflow-scoring-stream-app/lib

#shell
java -jar spring-cloud-dataflow-shell-1.2.3.RELEASE.jar 

#server
java -jar spring-cloud-dataflow-server-local-1.2.3.RELEASE.jar --spring.datasource.url=jdbc:mysql://172.17.0.1:3306/spring-cloud-cs-data-flow --spring.datasource.username=root --spring.datasource.password=password --spring.datasource.driver-class-name=org.mariadb.jdbc.Driver --spring.rabbitmq.host=127.0.0.1 --spring.rabbitmq.port=5672 --spring.rabbitmq.username=guest –spring.rabbitmq.password=guest

#DSL Stream example in shell
dataflow: stream create --definition "file | filter | log" -- name stream1

dataflow: stream create --definition "file--directory=/temp |filter |log" -- name ps-stream

#working with alias names
dataflow: stream create --definition ":ps-stream.alias>log"-- name ps-stream2

#open spring boot dataflow starters and grab rabbitMQ maven
https://cloud.spring.io/spring-cloud-stream-app-starters/

 Maven	https://dataflow.spring.io/rabbitmq-maven-latest

#goto dataflow shell and import rabbitmq app
app import --uri https://dataflow.spring.io/rabbitmq-maven-latest

dataflow: app list
dataflow:> app info sink:file

#create dataflow stream
dataflow:>stream create pluralsight-spring-cloud-cs --definition 'http --port=8086 | file --directory=/home/osboxes/temp/pluralsight/spring-cloud/dataflow/dataflow-out --suffix=txt --name=output'

dataflow:>stream list
dataflow:>stream deploy pluralsigth-spring-cloud-cs
Deployment request has been sent for stream 'pluralsight-spring-cloud-cs'

#create dataflow stream for batch files
dataflow:>stream create filestream1 --definition "file --directory=/home/osboxes/temp/pluralsight/spring-cloud/dataflow/dataflow-in --mode=lines | transform --expression=payload+' | processed=true' | output: file --directory=/home/osboxes/temp/pluralsight/spring-cloud/dataflow/dataflow-out --sufix=txt --name=output2"
datalow:>stream deploy filestream1

#create spring cloud task app
#take link from https://cloud.spring.io/spring-cloud-task-app-starters/
dataflow:> app import --uri https://dataflow.spring.io/task-maven-latest
dataflow:> app list
dataflow:> task create demotask --definition "timestamp --format=\MM-yy\"
Created new task 'demotask'
dataflow:>tast list


#create new stream via UI
http://localhost:9393/dashboard/index.html#/streams/create
toll-stream=http --port=8086 | split-JSON: splitter --expression=#jsonPath(payload,'$.station.readings') --spring.cloud.bindings.output.contentType='application/json' | log

#deploy it with parameters
deployer.*.local.inheritLogging=true


#create another stream via UI
http://localhost:9393/dashboard/index.html#/streams/create
toll-station-counter=:toll-stream.split-JSON > counter

#does not work another counter match is for property staationid
toll-stationid-counter=:toll-stream.split-JSON > field-value-counter --name=counter --field-name=stationid --spring.cloud.stream.bindings.input.contentType='application/json'


#create project spring-cloud-dataflow-scoring-app and create files, add mvn install to project and enter directory
cd /home/osboxes/eclipse-workspace/spring-cloud-coordinating-services-ch07-dataflow-scoring-stream-app/target


#register app to dataflow
dataflow:> app register --name fraudcheck --type processor --uri maven://com.milko.training.spring.cloud:spring-cloud-coordinating-services-ch07-dataflow-scoring-stream-app:jar:0.0.1-SNAPSHOT


dataflow:> app info processor:fraudcheck

#CREATE NEW STREAM via UI
http://localhost:9393/dashboard/index.html#/streams/create
toll-intake-stream=http --port=8086 | split-JSON: splitter --expression=#jsonPath(payload,'$.station.readings') --outputType=application/json | fraudcheck --outputType=application/json --fraud-thresshold=25 | log --level=INFO

#deploy it with parameters
deployer.*.local.inheritLogging=true


#deploy toll-intake-stream with partitioning parameters
deployer.fraudcheck.count=2
app.fraudcheck.producer.partitionKeyExpression=payload.stationid


#register app to task1 app
dataflow:> app register --name task1 --type task --uri maven://com.milko.training.spring.cloud:spring-cloud-coordinating-services-ch07-dataflow-task1:jar:0.0.1-SNAPSHOT

dataflow:>app info task:task1



#BULK DEFINE TASK
http://localhost:9393/dashboard/index.html#/tasks/bulk-define-tasks

bulktolldata=task1 --destination-file-path=dpath1 --source-file-path=spath1 --control-message=success
bulktolldata2=task1 --destination-file-path=dpath2 --source-file-path=spath2 --control-message=fail

#register task2 and task3
app register --name task2 --type task --uri maven://com.milko.training.spring.cloud:spring-cloud-coordinating-services-ch07-dataflow-task2:jar:0.0.1-SNAPSHOT
dataflow:>app info task:task2


app register --name task3 --type task --uri maven://com.milko.training.spring.cloud:spring-cloud-coordinating-services-ch07-dataflow-task3:jar:0.0.1-SNAPSHOT
dataflow:>app info task:task3

#CREATE COMPOSITE TASK
http://localhost:9393/dashboard/index.html#/tasks/create-composed-task

Composed1:
task1 --destination-file-path=path2 --control-message=ok --source-file-path=path1 && task2 && task3
#CREATE composed1

Composed2:
task1 --destination-file-path=dpath --control-message=fail --source-file-path=spath 'FAILED' -> task2 '*' -> task3

composite_mix
<task1 || task2> && <task3 || timestamp>


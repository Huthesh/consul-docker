#Consul-docker end to end orchestration 

This wiki explains about setting up end to end docker deployment, including service discovery, HAProxy load balancing, Distributed configuration store using consul.

Follow these steps:      
#####1. Run  consul server container
```
 docker run --rm --net=host  -h node \
 	-p 8500:8500 -p 53:8600/tcp -p 53:8600/udp \
  	consul agent -server \
  	-bind=<external IP address>  \
  	-client=<Client IP Address> -bootstrap -ui
```
If the container is started successfully we should be able to access Consul dashboard @ http://\<Client IP Address\>:8500/ and "dig @\<external IP address\> -p 8600 consul.service.consul" should display the DNS information
```
Mac does not have network mode as host, and external IP should be mapped to 0.0.0.0. UI will be accessible http://localhost:8500/
```
+ Consul should run in host network mode.     
+ consul agent -server will start the Consul in server mode       
+ -bind=<External IP address> IP Address used for Consul Cluster     
+ -client=<Client IP Address> Expose Consul interfaces on this IP address. You can access dash board http://\<Client IP Address\>:8500/  
  
+ -bootstrap : To run the server in leaser mode. Only one Server in cluster can run in this mode   
+ -ui : Enables Consul dashboard. Otherwise Consul Dash board will not be available   

##### 2. Install [Registrator](http://gliderlabs.com/registrator/latest/user/quickstart/)      
Registrator listen to Docker container start and stop events. And registers and deregisters the services in Cosul, when container is stopped or started.
```
docker run -d --name=registrator --net=host \
	--volume=/var/run/docker.sock:/tmp/docker.sock  \
	gliderlabs/registrator:latest \
	consul://<Consul external ip>:8500
```
######2.a Verify Registrator
  Check the Registrator log
```
#  docker logs registrator
2016/06/25 05:14:51 Starting registrator v7 ...
2016/06/25 05:14:51 Using consul adapter: consul://10.127.117.178:8500
2016/06/25 05:14:51 Connecting to backend (0/0)
2016/06/25 05:14:51 consul: current leader  10.127.117.178:8300
2016/06/25 05:14:51 Listening for Docker events ...
2016/06/25 05:14:51 Syncing services on 2 containers
2016/06/25 05:14:51 ignored: f4a6e7c9ea89 no published ports
2016/06/25 05:14:51 added: aa99be6606ec ubuntu:consul:8600:udp
2016/06/25 05:14:51 added: aa99be6606ec ubuntu:consul:8500
2016/06/25 05:14:51 added: aa99be6606ec ubuntu:consul:8600

```

##### 3. Run REDIS Container to verify the Service Discovery
```
docker run -d -P --name=redis redis
```

Once the redis container is running run following command. If everything is fine,then redis should be listed in the response.

```
# curl http://<cosul ip>:8500/v1/catalog/services

{"consul":[],"consul-8500":[],"consul-8600":["udp"],"redis":[]}
```
At this stage we have service discovery is up and running. 

##### 4. HAProxy reload using Consul template






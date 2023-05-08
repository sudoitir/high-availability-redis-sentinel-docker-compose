# Highly Available Redis Docker Compose

#### For see Spring boot with High Available Redis Sentinel example go to this link:
https://github.com/sudoitir/spring-with-redis-sentinel

### How to init

<p>Open the terminal in the directory where the Docker Compose file is located and enter the following command:</p>

```docker compose up -d```
<p>This command will create and start the Redis containers defined in the Docker Compose file.</p>

<p>To check whether all the Redis containers are up and running, you can use the following command:</p>

```docker ps```
<p>This will list all the running containers, including the Redis containers.</p>

<p>To ensure that the Redis Master container is healthy, you can use the following command:</p>

```docker inspect --format='{{json .State.Health}}' redis-master```
<p>This command will return the health status of the Redis Master container in JSON format. If the container is healthy,<br>
the output should be:</p>

```{"Status":"healthy"}```
<p>If the container is not healthy, the output will indicate the reason for the failure.</p>

<p>With these steps, you can check whether the Redis containers are up and running, and the Redis Master container is healthy.<br>
If everything is working correctly, your Redis setup should be ready to use.</p>

## Redis Master-Slave and Sentinel

<p> Redis is an open-source, in-memory key-value data store that is often used as a database, cache, or message broker.<br> 
Redis provides a mechanism called replication to replicate data from a Redis master to one or more Redis slaves.</p>

<p> When you set up Redis replication, one Redis instance acts as the master, and the other Redis instances act as slaves.<br>
The master Redis instance is responsible for receiving write commands from clients, while the slave Redis instances are<br>
responsible for replicating the data from the master and serving read requests from clients.</p>

<p>In the Docker Compose file, the Redis Master service is defined with the following command:</p>

```redis-server --appendonly yes ```
<p>This command starts the Redis server with the "appendonly" option enabled, which ensures that all write operations are<br>
written to disk. The Redis Slave service is defined with the following command:</p>

```redis-server --appendonly yes --slaveof redis-master 6379```
<p>This command starts the Redis server with the "slaveof" option, which makes the Redis Slave instance a slave of the<br> 
Redis Master instance.</p>

<p>Redis Sentinel is a high-availability solution for Redis that provides automatic failover in case the master Redis<br> 
instance fails. Sentinel monitors the Redis instances in a Redis replication setup and promotes a slave Redis instance<br>
to a master if the current master fails.</p>

<p>In the Docker Compose file, the Redis Sentinel service is defined with the following command:</p>

```redis-sentinel /usr/local/etc/redis/sentinel.conf```
<p>This command starts the Redis Sentinel service and specifies the location of the configuration file, <br>
which contains the Sentinel configuration options, such as the list of Redis instances to monitor and the criteria for failover.</p>

<p>In summary, Redis replication uses a master-slave model to replicate data between Redis instances, while Redis Sentinel<br>
provides automatic failover and high availability for Redis replication setups. Together, they provide a robust and<br>
scalable solution for building highly available Redis infrastructures.</p>

# Highly Available Redis Docker Compose With Consul And Spring Framework

### How to run

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

## Spring application part

<p>Add the Spring Cloud Consul dependency to your project's pom.xml file:</p>

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    <version>3.0.4</version>
</dependency> 
```

<p>Configure your Spring application to use Consul for service discovery by adding the following configuration to your application.yml file:</p>

```
spring:
  cloud:
    consul:
      host: consul
      port: 8500
      discovery:
        prefer-ip-address: true
```

<p>This configuration tells Spring Cloud Consul to use Consul for service discovery and to prefer IP addresses over host names.</p>

<p>Update your application code to use the Consul service registry to discover Redis Master's IP address and port number. Here's an example:</p>

```
@Configuration
@Slf4j
@RequiredArgsConstructor
@EnableRedisRepositories
public class RedisMasterService {

    private static final String REDIS_MASTER_SERVICE_ID = "redis-master";

    private final DiscoveryClient discoveryClient;


    public String getRedisMasterHost() {
        List<ServiceInstance> instances = discoveryClient.getInstances(REDIS_MASTER_SERVICE_ID);
        if (instances == null || instances.isEmpty()) {
            throw new IllegalStateException("Unable to locate Redis Master instance");
        }
        return instances.get(0).getHost();
    }

    public int getRedisMasterPort() {
        List<ServiceInstance> instances = discoveryClient.getInstances(REDIS_MASTER_SERVICE_ID);
        if (instances == null || instances.isEmpty()) {
            throw new IllegalStateException("Unable to locate Redis Master instance");
        }
        return instances.get(0).getPort();
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        log.info("Redis Template config using sentinel");
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new Jackson2JsonRedisSerializer<>(Object.class));
        return template;
    }

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        String redisMasterHost = getRedisMasterHost();
        int redisMasterPort = getRedisMasterPort();
        log.info("RedisConnectionFactory config using redis host: {}:{}", redisMasterHost, redisMasterPort);
        return new LettuceConnectionFactory(new RedisStandaloneConfiguration(redisMasterHost, redisMasterPort));
    }
}
```
<p>In this example, the RedisMasterService class uses the DiscoveryClient to obtain a list of service instances registered with<br>
Consul for the redis-master service. It then returns the host and port of the first service instance in the list.</p>

<p>Update your Redis configuration to use the Redis Master's IP address and port number obtained from the Consul service registry.<br>
Here's an example:</p>

```
spring:
  redis:
    host: ${redisMasterService.getRedisMasterHost()}
    port: ${redisMasterService.getRedisMasterPort()}
```
<p>In this example, the spring.redis.host and spring.redis.port properties are resolved at runtime using SpEL expressions<br>
that invoke the methods on the RedisMasterService bean to obtain the Redis Master's IP address and port number.</p>

<p>With these changes, your Spring application should be able to discover the Redis Master service using Consul and<br>
use its IP address and port number for Redis operations.</p>



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
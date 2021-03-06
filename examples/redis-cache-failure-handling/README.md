# Cache Failure Handling Example

This sample Spring App demonstrates using Redis as a cache for a time-intensive method. This could include lookup in another data service, or another slow process such as page generation.
* Appropriate if you have a method that will benefit from caching:
  - the method is slow to run
  - the method always returns the same result for a given input (similar to a pure function, but may have idempotent side effects)
  - the method is called repeatedly with the same input
* Appropriate if your App can handle a short period of time when the cache is unavailable
  - Redis for PCF service instances become unavailable for a few minutes during upgrades
* Not appropriate if you require direct access and manipulation of the underlying Redis cache

---

The sample app takes requests at `/token?id=<something>` and returns a token.
When Redis is missing or does not have the id cached, it creates a token. When Redis comes back up, continue to use the cache.

![Process Diagram](/assets/process_diagram.svg "Process Diagram")

## Run locally

Start a local redis-server:
```
redis-server
```
Then start the Spring app:
```
cd redis-cache-failure-handling
gradle bootRun
```

Visit in your browser:
```
http://localhost:8080/token?id=<ANY_ID>
```
Observe that the `duration` field of the response is long the first time each token is passed. This decreases drastically with subsequent calls when the Redis cache is used.


## Run on CF
Build the project:
```
gradle build
```
Push the app by providing the path to the app jar file:
```
cf push <APP_NAME> -p build/libs/redis-cache-failure-handling*.jar
```
This will print the route to the App, usually 'apps.<CF_URL>.com'.


Create a redis service. Spring auto-configures Redis connection using the service-binding so it should just work.
```
#cf create-service <REDIS_SERVICE> <SERVICE_PLAN> <SERVICE_INSTANCE_NAME>
 cf create-service  p.redis         cache-small   <SERVICE_INSTANCE_NAME>
```

Bind your redis service, and restage the App:
```
cf bind-service <APP_NAME> <SERVICE_INSTANCE_NAME>
cf restage <APP_NAME>
```

Reference: https://docs.cloudfoundry.org/buildpacks/java/configuring-service-connections/spring-service-bindings.html#redis

## Errors
* `RedisCacheConfig` and `RedisCacheError` intercept the errors when Redis is missing. This allows the request to fall through to the expensive token generation call.

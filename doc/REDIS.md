Add Redis client
---------------------------
  1. `telnet 127.0.0.1 6379`
  1. type `ping` and return shows '+PONG'. It works, then `quit`
  1. `docker exec -it php composer require symfony-bundles/redis-bundle`
  1. Edit `app/.env` and replace entry `REDIS_URL=tcp://redis:6379/0`
  1. Redis admin UI (Redis Commander): http://localhost:8081
  1. Implementation looks like
```php
use SymfonyBundles\RedisBundle\Redis as Redis;
public function myFunction()
{
    $redis = new Redis\Client([$_ENV['REDIS_URL']]);
}
```

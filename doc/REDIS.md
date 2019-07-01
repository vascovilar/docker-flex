Add Redis client
---------------------------
  1. `telnet 127.0.0.1 6379`
  1. `ping` getting +PONG check it works, then `quit`
  1. `composer require symfony-bundles/redis-bundle`
  1. Edit `.env` and replace entry `REDIS_URL=tcp://redis:6379/0`
  1. Implementation looks like
```php
use SymfonyBundles\RedisBundle\Redis as Redis;
public function my-function()
{
    $redis = new Redis\Client([$_ENV['REDIS_URL']]);
}
```
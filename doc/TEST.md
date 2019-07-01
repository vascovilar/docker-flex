Add functional test
---------------------------
  1. `composer require --dev symfony/phpunit-bridge`
  2. `php bin/console make:functional-test`
  3. `nano tests/ClientTest.php`
```php
$client = static::createClient();
$crawler = $client->request('GET', '/client/');
$this->assertSame(200, $client->getResponse()->getStatusCode());
$this->assertContains('Client index', $crawler->filter('h1')->text());
```
  4. `./bin/phpunit`
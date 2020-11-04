Add functional test
---------------------------
  1. `docker exec -it php composer require --dev symfony/phpunit-bridge`
  1. `docker exec -it php bin/console make:functional-test` with controller `Unicorn`
  1. Edit `app/tests/UnicornTest.php`
```php
        $client = static::createClient();
        $crawler = $client->request('GET', '/unicorn/');
        $this->assertSame(200, $client->getResponse()->getStatusCode());
        $this->assertContains('Unicorn index', $crawler->filter('h1')->text());
```
  4. `docker exec -it php bin/console doctrine:database:create --env=test`
  5. `docker exec -it php bin/console doctrine:schema:update --env=test --force`
  6. `docker exec -it php bin/phpunit`

Examples
---------------------------
```php
/* app/src/tests/ApiTestCase */

namespace App\Tests;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;
use Symfony\Component\HttpFoundation\Response;
use Hautelook\AliceBundle\PhpUnit\RefreshDatabaseTrait;

class ApiTestCase extends WebTestCase
{
    /*
     * Reload fixtures then revert at the end
     */
    use RefreshDatabaseTrait;

    protected $web;
    protected static $token;

    /**
     * For each test run this.
     */
    protected function setUp()
    {
        parent::setUp();
        $client = static::createClient();

        /* get token once */
        if (empty(static::$token)) {    
            $client->request(
                'POST',
                'api/login_check',
                [],
                [],
                ['CONTENT_TYPE' => 'application/json'],
                json_encode([
                  'username' => 'admin',
                  'password' => 'admin',
                ])
            );
            $data = json_decode($client->getResponse()->getContent(), true);
            static::$token = $data['token'];
        }

        /* create client */
        $this->web = $client;
        $this->web->setServerParameter('HTTP_Authorization', sprintf('Bearer %s', static::$token));
    }

    /**
     * Make an HTTP call to router.
     */
    protected function request(string $method, string $uri, $content = null, array $headers = []): Response
    {
        $server = ['CONTENT_TYPE' => 'application/json', 'HTTP_ACCEPT' => 'application/json'];
        foreach ($headers as $key => $value) {
            if ('content-type' === strtolower($key)) {
                $server['CONTENT_TYPE'] = $value;
                continue;
            }
            $server['HTTP_'.strtoupper(str_replace('-', '_', $key))] = $value;
        }
        if (is_array($content) && false !== preg_match('#^application/(?:.+\+)?json$#', $server['CONTENT_TYPE'])) {
            $content = json_encode($content);
        }
        $this->web->request($method, $uri, [], [], $server, $content);

        return $this->web->getResponse();
    }

    /**
     * Find entity in database.
     */
    protected function findOneEntity(string $className, array $search)
    {
        $repo = static::$container->get('doctrine')->getRepository($className);

        return $repo->findOneBy($search);
    }
}
```

```php
/* app/src/tests/Controller/UnicornControllerTest */
<?php

namespace App\Test\Controller;

use App\Entity\Unicorn;
use App\Repository\UnicornRepository;
use Symfony\Bundle\FrameworkBundle\KernelBrowser;
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class UnicornControllerTest extends WebTestCase
{
    private KernelBrowser $client;
    private UnicornRepository $repository;
    private string $path = '/unicorn/';

    protected function setUp(): void
    {
        $this->client = static::createClient();
        $this->repository = static::getContainer()->get('doctrine')->getRepository(Unicorn::class);

        foreach ($this->repository->findAll() as $object) {
            $this->repository->remove($object, true);
        }
    }

    public function testIndex(): void
    {
        $crawler = $this->client->request('GET', $this->path);

        self::assertResponseStatusCodeSame(200);
        self::assertPageTitleContains('Unicorn index');

        // Use the $crawler to perform additional assertions e.g.
        // self::assertSame('Some text on the page', $crawler->filter('.p')->first());
    }

    public function testNew(): void
    {
        $originalNumObjectsInRepository = count($this->repository->findAll());

        $this->markTestIncomplete();
        $this->client->request('GET', sprintf('%snew', $this->path));

        self::assertResponseStatusCodeSame(200);

        $this->client->submitForm('Save', [
            'unicorn[name]' => 'Testing',
            'unicorn[birthday]' => 'Testing',
        ]);

        self::assertResponseRedirects('/unicorn/');

        self::assertSame($originalNumObjectsInRepository + 1, count($this->repository->findAll()));
    }

    public function testShow(): void
    {
        $this->markTestIncomplete();
        $fixture = new Unicorn();
        $fixture->setName('My Title');
        $fixture->setBirthday('My Title');

        $this->repository->add($fixture, true);

        $this->client->request('GET', sprintf('%s%s', $this->path, $fixture->getId()));

        self::assertResponseStatusCodeSame(200);
        self::assertPageTitleContains('Unicorn');

        // Use assertions to check that the properties are properly displayed.
    }

    public function testEdit(): void
    {
        $this->markTestIncomplete();
        $fixture = new Unicorn();
        $fixture->setName('My Title');
        $fixture->setBirthday('My Title');

        $this->repository->add($fixture, true);

        $this->client->request('GET', sprintf('%s%s/edit', $this->path, $fixture->getId()));

        $this->client->submitForm('Update', [
            'unicorn[name]' => 'Something New',
            'unicorn[birthday]' => 'Something New',
        ]);

        self::assertResponseRedirects('/unicorn/');

        $fixture = $this->repository->findAll();

        self::assertSame('Something New', $fixture[0]->getName());
        self::assertSame('Something New', $fixture[0]->getBirthday());
    }

    public function testRemove(): void
    {
        $this->markTestIncomplete();

        $originalNumObjectsInRepository = count($this->repository->findAll());

        $fixture = new Unicorn();
        $fixture->setName('My Title');
        $fixture->setBirthday('My Title');

        $this->repository->add($fixture, true);

        self::assertSame($originalNumObjectsInRepository + 1, count($this->repository->findAll()));

        $this->client->request('GET', sprintf('%s%s', $this->path, $fixture->getId()));
        $this->client->submitForm('Delete');

        self::assertSame($originalNumObjectsInRepository, count($this->repository->findAll()));
        self::assertResponseRedirects('/unicorn/');
    }
}
```
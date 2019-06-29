# Docker-flex project

A simple LAMP Docker wrapper for Symfony 4 / Flex
- L : Ubuntu desktop
- A : Nginx http server
- M : Mariadb database server
- P : Php language
- +phpmyadmin


Requisites
-------------

  1. `sudo apt upgrade`
  1. `sudo apt update`
  1. `sudo apt install php7.2`
  1. `sudo apt install php7.2-xml`
  1. `sudo apt install php7.2-curl`
  1. `sudo apt install php7.2-mysql`
  1. `sudo apt install php7.2-mbstring`
  1. `sudo apt remove docker docker-engine docker.io containerd runc`
  1. `sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common`
  1. `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
  1. `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"`
  1. `sudo apt install docker-ce docker-ce-cli containerd.io`
  1. `sudo usermod -aG docker $USER`
  1. Edit `/etc/hosts` and add " db" (with a space before) at end of 127.0.0.1 line
  1. `sudo apt install composer`
  1. `sudo apt install docker-compose`
  1. `sudo apt install git`
  1. `sudo apt install nano`
  
  
1 - Create symfony skelton
---------------------------
  1. `cd ~`
  1. `composer create-project symfony/website-skeleton my-project-name` 
  1. `cd my-project-name`
  
2 - Add docker-flex project files
---------------------------
  1. `git init`
  1. `git remote add origin git@github.com:vascovilar/docker-flex.git`
  1. `git pull origin master`
  1. `rm -rf .git`
  
3 - Edit symfony conf
---------------------------
  1. Edit `.env` and replace entry `DATABASE_URL=mysql://root:root@db:3306/my-db-name`
  
4 - Start docker containers
---------------------------
  1. `docker-compose up -d`
  1. `docker-compose ps` to check all VM are Up 
  1. `php bin/console doctrine:database:create` to check symfony components are functional  
  
5 - Browse in chrome
---------------------------
  1. open app in a browser `localhost`
  1. open phpmyadmin in a browser `localhost:8080` and connect with user `root` and pass `root`
  
Make a CRUD page in 5 min
---------------------------
  1. `php bin/console make:entity`
  1. `php bin/console make:crud`
  1. `php bin/console make:migration`
  1. `php bin/console doctrine:migrations:migrate`
   
Fill with random data
---------------------------
  1. `composer require --dev hautelook/alice-bundle `
  1. `nano fixtures/client.yml`
```html 
App\Entity\Client:
	client_{1..10}:
		name: <name()>
		birthday: <dateTime()>
```
  3. `php bin/console hautelook:fixtures:load`
    
Add functional test
---------------------------
  1. `composer require --dev symfony/phpunit-bridge`
  1. `php bin/console make:functional-test`
  1. `nano tests/ClientTest.php`
```html 
$client = static::createClient();
$crawler = $client->request('GET', '/client/');
$this->assertSame(200, $client->getResponse()->getStatusCode());
$this->assertContains('Client index', $crawler->filter('h1')->text());
```
  4. `./bin/phpunit`
    
Create your own repo from this directory !
---------------------------
  1. `git init`
  1. `git add .`
  1. `git commit -m "Initial commit"`
 

Add JWT authentification
---------------------------
  1. `php bin/console make:entity` class `Auth` with `username:string:25`, `password:string:500`, `isactive:boolean`
  2. Edit `src/Entity/Auth.php` and modify like this:
```php
use Symfony\Component\Security\Core\User\UserInterface;
class Auth implements UserInterface
$username@ORM\Column(type="string", length=25, unique=true)
public function getSalt()
{
    return null;
}
public function getRoles()
{
    return array('ROLE_USER');
}
public function eraseCredentials()
{
}
```
  3. `php bin/console make:migration`
  4. `php bin/console doctrine:migrations:migrate`

  5. Edit `config/packages/security.yaml` and modify like this:
```yaml
security:
    encoders:
        App\Entity\Auth:
            algorithm: bcrypt
    providers:
        users:
            entity:
                class: App\Entity\Auth
                property: username
```
  6. `composer require lexik/jwt-authentication-bundle`
  7. `mkdir -p config/jwt`
  8. `openssl genrsa -out config/jwt/private.pem -aes256 4096` with pathphrase `de8dc53ffb0eac91d5881dfe1940fc8e`
  9. `openssl rsa -pubout -in config/jwt/private.pem -out config/jwt/public.pem`
  10. `chmod -R 644 config/jwt/*`
  11. Edit `config/packages/lexik_jwt_authentication.yaml`
```yaml  
lexik_jwt_authentication:
    secret_key: '%env(resolve:JWT_SECRET_KEY)%'
    public_key: '%env(resolve:JWT_PUBLIC_KEY)%'
    pass_phrase: '%env(JWT_PASSPHRASE)%'
    token_ttl: '%env(JWT_TOKEN_TTL)%'
```
  12. Edit `.env`
```yaml  
JWT_SECRET_KEY=%kernel.project_dir%/config/jwt/private.pem
JWT_PUBLIC_KEY=%kernel.project_dir%/config/jwt/public.pem
JWT_PASSPHRASE=de8dc53ffb0eac91d5881dfe1940fc8e
JWT_TOKEN_TTL=3600
```
  13. Edit `config/packages/security.yaml`
```yaml    
login:
    pattern:  ^/api/login
    stateless: true
    anonymous: true
    json_login:
        check_path: /api/login_check
        success_handler: lexik_jwt_authentication.handler.authentication_success
        failure_handler: lexik_jwt_authentication.handler.authentication_failure
api:
    pattern:   ^/api
    stateless: true
    guard:
        authenticators:
            - lexik_jwt_authentication.jwt_token_authenticator
dev:
    pattern: ^/(_(profiler|wdt)|css|images|js)/
    security: false
main:
    anonymous: true
``` 
  7. Edit config/routes.yaml
```yaml 
api_login_check:
    path: /api/login_check
    methods: ['POST']
``` 
  14. `php bin/console lexik:jwt:check-config`  
  15. `php bin/console security:encode-password` avec `admin`:`admin` coller dans table `Auth` l'utilisateur `admin` avec son mot de passe encodé et `is_active` à 1
  16. `curl -X POST -H "Content-Type: application/json" http://localhost/api/login_check -d '{"username":"admin","password":"admin"}'` 
  17. You'll receive something like that
```json
{
   "token" : "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXUyJ9.eyJleHAiOjE0MzQ3Mjc1MzYsInVzZXJuYW1lIjoia29ybGVvbiIsImlhdCI6IjE0MzQ2NDExMzYifQ.nh0L_wuJy6ZKIQWh6OrW5hdLkviTs1_bau2GqYdDCB0Yqy_RplkFghsuqMpsFls8zKEErdX5TYCOR7muX0aQvQxGQ4mpBkvMDhJ4-pE4ct2obeMTr_s4X8nC00rBYPofrOONUOR4utbzvbd4d2xT_tj4TdR_0tsr91Y7VskCRFnoXAnNT-qQb7ci7HIBTbutb9zVStOFejrb4aLbr7Fl4byeIEYgp2Gd7gY"
}
```
  18. Then send HTTP requests with header `Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXUyJ9.eyJleHAiOjE0MzQ3Mjc1MzYsInVzZXJuYW1lIjoia29ybGVvbiIsImlhdCI6IjE0MzQ2NDExMzYifQ.nh0L_wuJy6ZKIQWh6OrW5hdLkviTs1_bau2GqYdDCB0Yqy_RplkFghsuqMpsFls8zKEErdX5TYCOR7muX0aQvQxGQ4mpBkvMDhJ4-pE4ct2obeMTr_s4X8nC00rBYPofrOONUOR4utbzvbd4d2xT_tj4TdR_0tsr91Y7VskCRFnoXAnNT-qQb7ci7HIBTbutb9zVStOFejrb4aLbr7Fl4byeIEYgp2Gd7gY`
  
  
  

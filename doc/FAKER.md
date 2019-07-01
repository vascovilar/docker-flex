Fill with random data
---------------------------
  1. `composer require --dev hautelook/alice-bundle `
  2. Edit `config/packages/dev/nelmio_alice.yaml` and add new entry:
```yaml
nelmio_alice:
    locale: 'fr_FR'
```
  3. `nano fixtures/client.yml`
```yaml
App\Entity\Client:
    client_{1..10}:
        name: <name()>
        birthday: <dateTime()>
```
  4. `php bin/console hautelook:fixtures:load`
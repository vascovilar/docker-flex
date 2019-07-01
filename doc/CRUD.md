Make a CRUD page in 5 min
---------------------------
  1. `php bin/console make:entity`
  1. `php bin/console make:crud`
  1. `php bin/console make:migration`
  1. `php bin/console doctrine:migrations:migrate`
  
Add simple material design theme
---------------------------
  1. Edit `templates/base.html.twig` and add these lines to end of `<head>` tag:
```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/css/materialize.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/js/materialize.min.js"></script>
<style type="text/css">body{margin:20px;} </style>
```  
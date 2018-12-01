# Ferramentas:

* [Churn-PHP](https://github.com/bmitch/churn-php) - Para descoberta dos principais pontos de refatoração baseados no número de alterações que a classe sofre e na sua complexidade. 
```
    composer require bmitch/churn-php --dev
    vendor/bin/churn run ./src
```

* [PHP Testability](https://github.com/edsonmedina/php_testability) - Para conferencia de qualidade de código olhando a facilidade de criar testes para o código.
```
    composer require edsonmedina/php_testability "dev-master" --dev
    vendor/bin/testability
```

* [PhpCodeFixer](https://github.com/wapmorgan/PhpCodeFixer) - Para conferir compatibilidade do código em diferentes versões do php
```
    composer require wapmorgan/php-code-fixer dev-master --dev
    vendor/bin/phpcf ./app
```

* [PHP Code Sniffer](https://github.com/squizlabs/PHP_CodeSniffer) - Para manter o padrão de escrita do código, no nosso caso no padrão PSR2

* [PHP Mess Detector](https://github.com/phpmd/phpmd)

```
    ~/.composer/vendor/bin/phpmd app/ text unusedcode
```

[Agradecimneto à dseguy](https://github.com/exakat/php-static-analysis-tools#bugs-finders)
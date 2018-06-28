# Protected Money

[![Build Status](https://img.shields.io/travis/programarivm/protected-money.svg?style=flat-square)](https://travis-ci.org/programarivm/protected-money)
![Money PHP](/resources/logo.png?raw=true)

This is a fork of [moneyphp/money](https://github.com/programarivm/protected-money) that removes the BC Math and GMP functions. The API remains the same as in the [official documentation](http://moneyphp.org/en/latest/).

## Install

Via Composer

```bash
$ composer require programarivm/protected-money
```

## Why Remove the BC Math and the GMP Extensions?

> The short answer: to reduce the attack surface of your app.

`moneyphp/money` assumes that very huge amounts of money are going to be handled by default, considering that you are bypassing the limit established by `PHP_INT_MAX`. More specifically, the `Money` class internally picks [BC Math](http://php.net/manual/en/book.bc.php) or [GMP](http://php.net/manual/en/book.gmp.php) by default if detecting those PHP extensions, as it is shown next.

### `src/Money.php`

```php
<?php

namespace Money;

use Money\Calculator\BcMathCalculator;
use Money\Calculator\GmpCalculator;
use Money\Calculator\PhpCalculator;

/**
 * Money Value Object.
 *
 * @author Mathias Verraes
 */
final class Money implements \JsonSerializable
{
    ...

    /**
     * @var array
     */
    private static $calculators = [
        BcMathCalculator::class,
        GmpCalculator::class,
        PhpCalculator::class,
    ];

    ...

    /**
     * @return Calculator
     *
     * @throws \RuntimeException If cannot find calculator for money calculations
     */
    private static function initializeCalculator()
    {
        $calculators = self::$calculators;
        foreach ($calculators as $calculator) {
            /** @var Calculator $calculator */
            if ($calculator::supported()) {
                return new $calculator();
            }
        }
        throw new \RuntimeException('Cannot find calculator for money calculations');
    }

    ...
}
```

In short, BC Math and GMP are encouraged to be installed. However, arbitrary precision arithmetic functions should not be used in common web apps; according to [the least privilege principle](https://programarivm.com/the-least-privilege-principle-applied-to-php-bigints/) those extensions should be installed only if necessary.

## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.

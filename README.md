# Paggern

[![Build Status](https://travis-ci.org/gajus/paggern.png?branch=master)](https://travis-ci.org/gajus/paggern)
[![Coverage Status](https://coveralls.io/repos/gajus/paggern/badge.png?branch=master&cache=123)](https://coveralls.io/r/gajus/paggern?branch=master)
[![Latest Stable Version](https://poser.pugx.org/gajus/paggern/version.png)](https://packagist.org/packages/gajus/paggern)
[![License](https://poser.pugx.org/gajus/paggern/license.png)](https://packagist.org/packages/gajus/paggern)

Pattern interpreter for generating random strings.

## Generator

```php
$generator = new \Gajus\Paggern\Generator();

/**
 * Generate a set of random codes based on Paggern pattern.
 * Codes are guaranteed to be unique within the set.
 *
 * @param string $pattern Paggern pattern.
 * @param int $amount Number of codes to generate.
 * @param int $safeguard Number of additional codes generated in case there are duplicates that need to be replaced.
 * @return array
 */
$codes = $generator->generateFromPattern('FOO[A-Z]{10}[0-9]{2}', 100);
```

The above example will generate an array containing 100 codes, each prefixed with "FOO", followed by 10 characters from "ABCDEFGHKMNOPRSTUVWXYZ23456789" haystack and 2 numbers from "0123456789" haystack.

Paggern utilises [RandomLib](https://github.com/ircmaxell/RandomLib) to generate the pattern matching random character pool.

## Lexer

Lexer exposes a method to tokenise the string. `Lexer` is independant of the `Generator`. You can choose to interpret `Lexer` tokens using your own implementation of the `Generator`.

```php
$lexer = new \Gajus\Paggern\Lexer();

/**
 * Tokeniser explodes input into components describing the properties expressed in the pattern.
 *
 * @param string $pattern
 * @param boolean $expand Augment token definition with the haystack of possible values.
 * @return array
 */
$lexer->tokenise('[a-c]{3}[1-3]{3}', true);
```

```php
[
    [
        'type' => 'range',
        'token' => 'a-c',
        'repetition' => 3,
        'haystack' => 'abc'
    ],
    [
        'type' => 'range',
        'token' => '1-3',
        'repetition' => 3,
        'haystack' => 123
    ]
]
```

### Supported Tokens

#### Literal

Pattern can consist of literal characters, e.g. prefix of suffix of the code.

```php
$lexer->tokenise('abc');
```

```php
[
    [
        'type' => 'literal',
        'string' => 'abc'
    ]
]
```

The above pattern commands that the string is literally "abc".

#### Range

Range can be either numeric or ASCII.

```php
$lexer->tokenise('[a-z]');
```

In the `[a-z]` example, string must be a character from "abcdefghijklmnopqrstuvwxyz" haystack.

```php
[
    [
        'type' => 'range',
        'token' => 'a-z',
        'repetition' => 1
    ]
]
```

#### Range with Repetition

If the character must occur more than once, use repetition.

```php
$lexer->tokenise('[a-c]{3}');
```

In the `[a-z]{3}` example, string must consist of 3 characters from the "abc" haystack.

```php
[
    [
        'type' => 'range',
        'token' => 'a-c',
        'repetition' => 3
    ]
]
```

#### Character Classes

Predefined character classes can be used instead of ranges.

|Character Class|Range|
|---|---|
|`\U`|"ABCDEFGHKMNOPRSTUVWXYZ23456789" (or A-Z0-9 excluding IJLQ01) describes characters that are unambiguously recognised regardless of the font or case-sensitivity. The designated use case is voucher codes.|

#### Character Classes with Repetition

Similar to the Range with Repetition, Character Classes can be used with repetition, e.g.

```php
$lexer->tokenise('\U{3}');
```

## Limitations

* Pattern cannot include `[]{}` characters.
* Pattern cannot include characters outside of the ASCII.
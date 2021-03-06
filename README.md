Multicache
=========

> Adds array caching to Laravels cache drivers and custom drivers.

[![Build Status](http://img.shields.io/travis/pulkitjalan/multicache.svg?style=flat-square)](https://travis-ci.org/pulkitjalan/multicache)
[![Scrutinizer Code Quality](http://img.shields.io/scrutinizer/g/pulkitjalan/multicache/master.svg?style=flat-square)](https://scrutinizer-ci.com/g/pulkitjalan/multicache/)
[![Coverage Status](https://img.shields.io/scrutinizer/coverage/g/pulkitjalan/multicache/master.svg?style=flat-square)](https://scrutinizer-ci.com/g/pulkitjalan/multicache/code-structure/master)
[![License](http://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](http://www.opensource.org/licenses/MIT)
[![Latest Version](http://img.shields.io/packagist/v/pulkitjalan/multicache.svg?style=flat-square)](https://packagist.org/packages/pulkitjalan/multicache)
[![Total Downloads](https://img.shields.io/packagist/dt/pulkitjalan/multicache.svg?style=flat-square)](https://packagist.org/packages/pulkitjalan/multicache)

## Requirements

- PHP >= 5.5.9
- Laravel = 5.1

Laravel 5.2 natively supports a caching multiple items.

## Installation

Install via [composer](https://getcomposer.org/) - edit your `composer.json` to require the package.

```js
"require": {
    "pulkitjalan/multicache": "0.3.*"
}
```

Then run `composer update` in your terminal to pull it in.

Now add the following to the `providers` array in your `config/app.php`

```php
PulkitJalan\Cache\Providers\MultiCacheServiceProvider::class
```

## Usage

Any existing cache drivers and custom drivers will have access to the following new methods:

```php
    /**
     * Determine if an array of items exists in the cache.
     *
     * @param  array  $keys
     * @return array
     */
    public function hasMany(array $keys);

    /**
     * Retrieve an array of items from the cache by keys.
     *
     * @param  array  $keys
     * @param  mixed  $default
     * @return array
     */
    public function getMany(array $keys, $default = null);

    /**
     * Retrieve an array of items from the cache and delete them.
     *
     * @param  array  $keys
     * @param  mixed  $default
     * @return array
     */
    public function pullMany(array $keys, $default = null);

    /**
     * Store an array of items in the cache.
     *
     * @param  array  $items
     * @param  \DateTime|int  $minutes
     * @return void
     */
    public function putMany(array $items, $minutes);

    /**
     * Store an array of items in the cache if the key does not exist.
     *
     * @param  array  $items
     * @param  \DateTime|int  $minutes
     * @return bool
     */
    public function addMany(array $items, $minutes);

    /**
     * Store an array of items in the cache indefinitely.
     *
     * @param  array  $items
     * @return void
     */
    public function foreverMany(array $items);

    /**
     * Get an array of items from the cache, or store the default value.
     *
     * @param  array  $keys
     * @param  \DateTime|int  $minutes
     * @param  \Closure  $callback
     * @return mixed
     */
    public function rememberMany(array $keys, $minutes, Closure $callback);

    /**
     * Get an array of items from the cache, or store the default value forever.
     *
     * @param  array  $keys
     * @param  \Closure  $callback
     * @return mixed
     */
    public function searMany(array $keys, Closure $callback);

    /**
     * Get an array of items from the cache, or store the default value forever.
     *
     * @param  array  $keys
     * @param  \Closure  $callback
     * @return mixed
     */
    public function rememberManyForever(array $keys, Closure $callback);

    /**
     * Remove an array of items from the cache.
     *
     * @param  array  $keys
     * @return bool
     */
    public function forgetMany(array $keys);
```

Most of the existing methods like `has`, `get`, `put`, `forget`... will also accept an array and automatically run the relevant `Many` function. As Expected the original methods will return results in the same format as they always have if called without an array.

## Examples

Below are a few examples of how to use the functions and what they return.

### Has

```php
$keys = [
    'one', // exists
    'two', // does not exist
    'three', // exists
];

Cache::hasMany($keys);

// or

Cache::has($keys);

// will return: ['one' => true, 'two' => false, 'three' => true]
```

### Get

```php
$keys = [
    'one', // exists
    'two', // does not exist
    'three', // exists
];

Cache::getMany($keys);

// or

Cache::get($keys);

// will return: ['one' => 'data', 'two' => null, 'three' => 'data']
```

### Put

The `put` method works a little differently to `putMany`. Where `putMany` accepts a key value array as the first parameter and the number of minutes to store for as the second parameter, the `put` method takes two separate arrays as the first two parameters and minutes as the third parameter.

Eg:

```php
$data = [
    'key1' => 'value1',
    'key2' => 'value2',
    'key3' => 'value3',
];

Cache::putMany($data, 10);

// or

Cache::put(array_keys($data), array_values($data), 10);
```

### Forget

```php
$keys = [
    'one',
    'two',
    'three',
];

Cache::forgetMany($keys);

// or

Cache::forget($keys);

// will return: ['one' => true, 'two' => true, 'three' => true]
```

## How does it work?

For any driver that does not have an underlying `Many` method, the methods will call the `Non-Many` version of the method for every item in the array.

For example, when using the `apc` driver, which does not offer its own `getMany` method, then the `get` method will be called ten times if there are ten items in the input array.

Now, when using the `memcached` driver, which does have its own `getMany` method, then the `getMany` method will be called once and the data returned.

Currently the `MemcachedStore`, `DatabaseStore`, `RedisStore` and the `ArrayStore` are the only ones to offer their own `Many` methods. **More to come soon...**

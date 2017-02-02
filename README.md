Gamer Network is hiring! http://jobs.gamesindustry.biz/gamer-network/

# EggCup

Caching 'decorators' for PHP classes.

Used to wrap any PHP object with a redis or memcached backed caching
layer.  Uses docstrings on 'hosted' object's methods to determine caching
behaviour (such as TTL, invalidation tags), obviating need to substantially refactor an application.

The Redis client supports efficient tag based cache invalidation.
Memcache attempts to emulate this using string based list emulation, which is
slow and not very atomic.  Only use invalidation if you really need it -
in many cases just letting keys expire will suit needs (we ran Eurogamer
for years with this model).

Memcache support requires one of the two PHP memcached modules (called 'Memcached' and
'Memcache' - note no 'd' in second. Yes it's confusing).
Memcached (with 'd') is recommended.

Redis mode (`Redis.inc.php`) supports both Predis (native PHP
version) and phpredis (C module), the latter being about 3 times faster.

### Examples

```php
class MyExistingClass extends \Eggcup\Cacheable {

    /**
    * cache-me
    * cache-expiry: 60
    */
    public function getSomeDataFromDB( $arg1, $arg2 ) {
        // take a long time to generate some data using $args1 and $args2 as determining vars.
        return $someData;
    }
}

$cachedclass = new \Eggcup\Redis( new MyExistingClass(), array( array( "host" => "192.168.4.142", "port" => "11216" ) ) );

// this return value will be cached for 60s
// first call will use DB
$cachedclass->getSomeDataFromDB( 1, 2 );

// second call will use memcache
$cachedclass->getSomeDataFromDB( 1, 2 );

// different args so will use DB again (cache key auto constructed from args)
$cachedclass->getSomeDataFromDB( 3, 4 );

sleep( 61 );

// will us DB again
$cachedclass->getSomeDataFromDB( 1, 2 );
```

Cache-invalidation is also possible by tagging methods::

```php
/**
* cache-me
* cache-expiry: 60
* cache-invalidate-on: tag1, tag2
*/
public function getSomeDataFromDB( $arg1, $arg2 ) {
    // take a long time to generate some data using $args1 and $args2 as determining vars.
    return $someData;
}
```

Typically tags relate to SQL table names.

Then simply tag methods that write data and they will invalidate all keys with these tags::

```php
/**
* cache-flush: tag1
*/
public function writeToSomeTable1( $data ) {
    //writes to some table
}
/**
* cache-flush: tag2
*/
public function writeToSomeTable2( $data ) {
    //writes to some table
}
```

Calling either of these method will delete all cache keys written by
methods tagged with either tag1, tag2 or both.

## Debugging

You can force all reads to miss by setting:

```php
define('CACHE_BYPASS');
```

Results of calls to database backend will still be written to cache.

## Limitations

Intraobject method calls, such as `$this->fetchMethod()` will not by default be cached and to do so requires a very small refactor to `$this->_cup->fetchMethod()`.

Note: **you must call `\Eggcup\Cacheable->__construct()` from your subclass's `__construct()` or `::_cup` will not be defined.

## Known Issues

There are no tests. :-#


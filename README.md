[![Author](https://img.shields.io/badge/author-Daniel%20M.%20Hendricks-lightgrey.svg?colorB=9900cc&style=flat-square)](https://www.danhendricks.com/?utm_source=github.com&utm_medium=campaign&utm_content=button&utm_campaign=wordpress-object-cache-helper)
[![GitHub License](https://img.shields.io/badge/license-GPLv2-yellow.svg?style=flat-square)](https://raw.githubusercontent.com/dmhendricks/wordpress-object-cache-helper/master/LICENSE)
[![Analytics](https://ga-beacon.appspot.com/UA-67333102-2/dmhendricks/wordpress-object-cache-helper?flat)](https://github.com/igrigorik/ga-beacon/?utm_source=github.com&utm_medium=referral&utm_content=button&utm_campaign=dmhendricks%2Fwordpress-object-cache-helper)
[![Get Flywheel](https://img.shields.io/badge/hosting-Flywheel-green.svg?style=flat-square&label=compatible&colorB=AE2A21)](https://share.getf.ly/e25g6k?utm_source=github.com&utm_medium=campaign&utm_content=button&utm_campaign=dmhendricks%2Fwordpress-object-cache-helper)
[![Twitter](https://img.shields.io/twitter/url/https/github.com/dmhendricks/wordpress-object-cache-helper.svg?style=social)](https://twitter.com/danielhendricks)

# WordPress Object Cache Wrapper Class

A simple [MU plugin](https://codex.wordpress.org/Must_Use_Plugins) for WordPress that acts as a wrapper for [WP Object Cache](https://codex.wordpress.org/Class_Reference/WP_Object_Cache) functions, with support for flush cache by group.

It was created as an MU plugin so that it is loaded and available for use in the theme as well as standard custom plugins, useful to cache heavy operations that fall outside of WordPress's built-in caching (such as [direct database queries](https://codex.wordpress.org/Class_Reference/wpdb), file and remote operations, etc).

:construction: This project is **beta** pending further testing and added features.

#### TODO

- Add ability to delete cache key stored in group

## Installation

Simply copy the `object-cache-helper.php` file to your `wp-content/my-plugins` directory (create one if it does not exist).

## Usage

### Instantiation

Without arguments:

```php
$cache = new \MU_Plugins\WP_Cache_Object();
```

With [arguments](#arguments):

```php
$cache = new \MU_Plugins\WP_Cache_Object([
   'expire' => HOUR_IN_SECONDS * 8,
   'group' => 'my_cache_group'
]);
```

### Getting/Setting Cache Value

In this example, we will retrieve the public IP address of the server from [SeeIP](https://seeip.org/) using an anonymous function as the callback and cache it for one day:

```php
function get_public_ip_address() {
  $result = wp_remote_get( 'https://ip4.seeip.org' );
  return isset( $result['body'] ) ? $result['body'] : null;
}

$cache = new \MU_Plugins\WP_Cache_Object( [ 'expire' => DAY_IN_SECONDS ] );
$ip_address = $cache->get_object( 'my_server_public_ip_address', 'get_public_ip_address' );

echo 'Public IP Address: ' . $ip_address;
```

Note that in this example, we have one line of code in the callback function, but you can add as much logic as you like.

#### Anonymous Function as Callback

In this example, we will do the same as above with an anonymous function as the callback:


```php
$cache = new \MU_Plugins\WP_Cache_Object( [ 'expire' => DAY_IN_SECONDS ] );

$ip_address = $cache->get_object( 'my_server_public_ip_address', function() {
  $result = wp_remote_get( 'https://ip4.seeip.org' );
  return isset( $result['body'] ) ? $result['body'] : null;
});

echo 'Public IP Address: ' . $ip_address;
```

#### Passing Variables to Callback Function

This example shows how to pass variables to an anonymous callback. You would not cache this in practice and purely serves as an example for passing variables. In this example, we pass `$name` and `$age` to the callback.

```php
$cache = new \MU_Plugins\WP_Cache_Object( [ 'expire' => HOUR_IN_SECONDS * 12 ] );

$name = 'Daniel';
$age = 29;

$greeting = $cache->get_object( 'about_me_string', function() use ( &$name, &$age ) {
  return sprintf( 'Hello %s. You are %d years old.', $name, $age );
});

echo $greeting; // Hello Daniel. You are 29 years old.
```

### Flushing the Cache

You can flush the entire cache using [`wp_cache_flush()`](https://developer.wordpress.org/reference/functions/wp_cache_flush/), or you can flush a _group_ that was created _using this class_ with the following:

```php
$cache->flush_group( 'my_cache_group' );
```

#### Deleting a Single Key from a Cache Group

```php
$cache->delete_group_key( 'my_cache_key_name' ); // Removes key from default group

$cache->delete_group_key( 'my_cache_key_name', 'custom_cache_group' ); // Removes key from specific group
```

## Arguments

The following arguments are supported when instantiating the class:

| **Option**       | **Description**                                                               | **Type** | **Default**             |
|------------------|-------------------------------------------------------------------------------|----------|-------------------------|
| `group`          | The group to store the cache key in                                           | string   | {theme_dir}_cache_group |
| `expire`         | Number of seconds to cache the value                                          | int      | 3600 (1 hour)           |
| `single`         | Store as single key rather than group array                                   | bool     | false                   |
| `network_global` | Set to true to store cache value for entire network, rather than per sub-site | bool     | false                   |
| `force`          | Always return uncached value, useful for debugging                            | bool     | false                   |

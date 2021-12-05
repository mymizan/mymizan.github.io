---
layout: default
title: "The right way to check if a plugin is active"
date: "2020-01-31"
categories: 
  - "life"
---

There are many different ways to check if a plugin is active on your WordPress website. However, there are pros and cons to each approach.

In this article, we will explore different ways to check if a plugin is active on a WordPress site and finally write a code snippet that will address all issues.

## Check for a function or class

Suppose, your plugin relies on **[wc\_get\_product](https://docs.woocommerce.com/wc-apidocs/function-wc_get_product.html)** to retrieve a WooCommerce product from the database. As you know WooCommerce plugin is most likely to define this function, you can make a check to see if this function is defined and if it is defined, you can assume that WooCommerce is active.

```
if ( function_exists('wc_get_product') ) {
  // WooCommerce is active, assumed.
}

if ( class_exists('WC_Product') ) {
  // WooCommerce is active, assumed.
}
```

However, in rare cases, it could be possible that another developer had mistakenly defined a similar function. So, checking with [function\_exists()](https://www.php.net/manual/en/function.function-exists.php) will lead to wrong results.

> Well, another problem with this method is, there's no gurantee that your depenent plugin will load first! What if WordPress loads your plugin before it loads your dependent plugin? If that happens, the above method will fail.

So, while this method is perfectly fine for checking if a particular function exists (say for backward compatibility), it isn't the right way to check if a plugin is active on a website. The best way to check whether a plugin is active or not is to use the WordPress methods.

## Check with is\_pluign\_active()

WordPress provides a function [is\_plugin\_active()](https://codex.wordpress.org/Function_Reference/is_plugin_active) (is\_plugin\_active\_for\_network() for multisite) to check if a particular plugin is active on the site. The function is defined only when you are on the WordPress backend and it needs to be on or after the admin\_init hook.

```
// make sure you are checking on the backend and referencing
// after admin_init hook is fired.
if ( is_plugin_active('woocommerce/woocommerce.php') ) {
  // Let your plugin activate or run your code.
}
```

## The Problem?

The main problem with is\_pluign\_active() is that it requires the path to the plugin's main file relative to the **plugins** directory. Normally, when WooCoomerce is installed using the plugin installer, it will be installed on _woocommerce_ directory and the path to the plugin, relative to **plugins** directory will be 'woocommerce/woocommerce.php'.

But what if users upload the files using FTP and name the directory other than woocommerce? Your call to is\_plugin\_active() will return false, even when the plugin is installed and active.

## What's the best way?

The best way would be a code snippet that addresses all the possible issues mentioned above.

```
/** 
 * Don't forget to set your prefix.
 */ 
function prefix_is_plugin_active( $plugin_main_file ) {
	// get the list of plugins.
	$active_plugins = apply_filters( 'active_plugins', get_option( 'active_plugins' ) );

	// escape characters that have special meaning in regex.
	$plugin_main_file = preg_quote( $plugin_main_file, '/' );
	$is_plugin_installed = false;

	// Loop through the active plugins.
	foreach( $active_plugins as $plugin ) {
	    if( preg_match( '/.+\/' . $plugin_main_file . '/', $plugin ) ) {
		$is_plugin_installed = true;
		break;
	    }
	}

	return $is_plugin_installed;
}
```

The method above will work pretty much well, except that it will fail for single-file and must use plugins. It also won't work for network activated plugins.

## Covering all situations

Below I wrote a class that will check all plugins: must use, single file and network activated (WordPress multisite). I won't explain the code, as I have put comments in the code. YOu can download it from my [GitHub repository](https://github.com/mymizan/wp-check-if-plugin-active) for your own use.

```
class Utils_Plugins {

	/**
	 * Check if a plugin is active
	 *
	 * @param string $plugin_main_file main file of the plugin, eg. woocommerce.php
	 * @return bool True if plugin is active, false otherwise. 
	 */
	public static function is_active( $plugin_main_file ) {
		// get the list of plugins.
        $active_plugins = apply_filters( 'active_plugins', get_option( 'active_plugins' ) );

        // escape characters that have special meaning in regex.
        $plugin_main_file = preg_quote( $plugin_main_file, '/' );
        $is_plugin_installed = false;

        // Loop through the active plugins.
        foreach( $active_plugins as $plugin ) {
            if( preg_match( '/.+\/' . $plugin_main_file . '/', $plugin ) ) {
                $is_plugin_installed = true;
                break;
            }
        }

        return $is_plugin_installed;
	}

	/**
	 * Check if a plugin is network active
	 *
	 * @param string $plugin_main_file main file of the plugin, eg. woocommerce.php
	 * @return bool True if plugin is active, false otherwise. 
	 */
	public static function is_network_active( $plugin_main_file ) {

		// if not a multisite, don't check.
		if ( ! is_multisite() ) {
	        return false;
	    }

		// get the list of plugins.
        $active_plugins =  get_site_option( 'active_sitewide_plugins' );

        // escape characters that have special meaning in regex.
        $plugin_main_file = preg_quote( $plugin_main_file, '/' );
        $is_plugin_active = false;

        // Loop through the active plugins.
        foreach( $active_plugins as $plugin_name => $plugin_activation ) {
            if( preg_match( '/.+\/' . $plugin_main_file . '/', $plugin_name ) ) {
                $is_plugin_active = true;
                break;
            }
        }

        return $is_plugin_active;
	}

	/**
	 * Check if a must use (mu) plugin exists.
	 *
	 * mu plugins are always active. So there's no need to check if they are 
	 * active or not. We just need to check that they are in the list.
	 *
	 * @param string $plugin_main_file main file of the plugin, eg. woocommerce.php
	 * @return bool True if plugin matches, false otherwise. 
	 */
	public static function is_mu_active( $plugin_main_file ) {
		$_mu_plugins = get_mu_plugins();

		if ( isset( $_mu_plugins[ $plugin_main_file ] ) ) {
			return true;
		}

		return false;
	}

}

```

## Final Solution

The final solution is a class with three static methods covering all cases. Though, the code will still fail for single-file plugins. I haven't fixed that intentionally as that's a rare case.

---
layout: default
title: "Redirect All or Specific Pages to Homepage"
date: "2019-08-23"
categories: 
  - "life"
---

WordPress has many complex plugins to handle redirection, such as [Redirection](https://wordpress.org/plugins/redirection/) by [John Godley](https://johngodley.com/).

However, in this tutorial, we will build a simple plugin that will redirect all or specific pages to the homepage.

You may use this plugin where a sophisticated plugin is unnecessary.

## Plugin Header

Lets first write the plugin header, the comment block that will make our file appear as a plugin to WordPress.

```
<?php
/*
Plugin Name: Simple Redirection
Plugin URI: https://developerhero.net
Description: This is a simple plugin that will redirect all or certain pages to the homepage. 
Author: M Yakub Mizan
Version: 1.0.0
Author URI: https://yakub.xyz
*/
```

Create a new file simple-redirection.php, copy the above code into it and then place it under your WordPress installation's plugin directory. Now, WordPress will treat this file as a plugin and you can see it in the plugins list.

## Code to Handle Redirection

Next, we will add the code that will redirect the users to the homepage. Please, notice the comment block to know where you need to make changes.

```
function mymizan_simple_redirection_plugin()
{
	/**
	* If the user is already on frontpage or home 
	* page (blogs) or in the admin area return false.
	*/
	if ( is_front_page() ||
 	     is_home() ||
 	     is_admin() ) {
		return false;
	}

	/**
	* Add the whitelisted page IDs (eg. 1, 2, 3, 4) here such as 
	* cart, checkout, shop, privacy policy etc. 
	* You don't need to add homepage, blog page as they are 
	* already being checked above
	*/
	$whitelist = array(
		//put page IDs to ignore here
	);

	/**
	* The page you want to redirect to
	*/
	$redirect_url = 'https://www.example.com/';

	$current_object_id = get_queried_object_id();

	if ( is_page() && ! in_array( $current_object_id, $whitelist ) ) {
		header( "Location: " . $redirect_url );
		die;
	}
}

add_action('wp', 'mymizan_simple_redirection_plugin');
```

Append the above code to the file you created in step one. Then activate the plugin from the plugins menu. Now, all pages, except those mentioned in the whitelist, will be redirected to the homepage.

## Download the Plugin

You can download the plugin from [GitHub](https://github.com/mymizan/simple-redirectioon-plugin).

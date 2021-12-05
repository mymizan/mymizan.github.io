---
layout: default
title: "Building a Simple WordPress Options Page"
date: "2019-06-01"
categories: 
  - "wordpress"
---

Options table (prefix\_options) is WordPress's way of providing a key-value store for plugin authors. It's a good place to save any key value data such as settings used by your plugin.

Every usable plugin will need to store some piece of data that is only related to the plugins. In most cases, the options table is the most suitable place to store such data.

In this tutorial, we will learn how to build an options page for your plugin and to store the options in the options table. Luckily it's very easy to build an options page with WordPress. Note that the options page is also known as settings page.

## First Build a Simple Plugin

We will put our code in a plugin. If you have never built one, don't be deterred by it. It's so easy to build a plugin with WordPress! To build a simple plugin, all we will need is a Docblock and a simple class to put our code into.

```
<?php
/*
Plugin Name: YM Options Page
Plugin URI: https://yakub.xyz/
Description: The plugin shows you how to build an options page
Author: M Yakub Mizan
Version: 1.0.0
Author URI: https://yakub.xyz
*/

class YM_Options_Page {

}

new YM_Options_Page();

```

That's exactly how much code we need to build a simple plugin. Copy the code above, save that in a file named _ym-options.php_ and put that in your plugin folder. Then head over to the plugin list and activate our new plugin.

We will put all of our code in this new plugin we have just created.

## Add a Menu Under Settings

You need a "link" somewhere to access your options page. In this section, we will deal with the link. The link can be a top-level menu or a submenu under any of the top-level menus in the WordPress admin dashboard. We are going to add the submenu under the settings.

## Link for the Settings Page

As said above, we need to put a link under settings section of your WordPress admin dashboard to access the page we are going to build.

To achieve that, we are going to define two methods _menu\_callback_ and _menu\_markup_, which will contain a call to _add\_options\_page_ function and the HTML markup of the page correspondingly. Our newly defined methods will look like below.

```
/**
 * Callback to call the function that will generate the menu link
 * We will hook it later. 
 */
public function menu_callback()
{
  add_options_page( 'YM Options Page', 'Ym Options', 'manage_options', 
            'ym-options', array($this, 'menu_markup') );
}

/**
 * Contains menu markup. We will extend it later. 
**/
public function menu_markup()
{
  if ( !current_user_can( 'manage_options' ) )  {
      wp_die( __( 'You do not have sufficient permissions to access this page.' ) );
}
?>
<div class="wrap">
  <h1>YM Options</h1>
</div>
<?php
}
```

Add these two methods to the class we have defined above. Then go ahead and define the constructor below. As shown in the constructor, which will be invoked when the plugin runs, we will call the _admin\_init_ to hook the _menu\_callback_ method we defined above.

```
/**
 * This is the place where we define our hooks.
 * They run when the class is instantiated. 
**/
public function __construct()
{
   /** hook the function that will call 
    * the add_options_page to add the options 
   **/
   add_action( 'admin_menu', array($this, 'menu_callback') );
}
```

You may ask, why can't we do call the _add\_options\_page_ directly in the constructor? Why do we need to hook into _admin\_init_ first to call the function?

Well, most things in WordPress are achieved via hooks. If you call the function directly, chances are that it will be called before the definition is loaded, leading to a fatal error, which is why we have to use hooks to call that function.

That's it! We are ready to see some result of our hard work! Save the file and then reload WordPress. Now click on YM Options under settings. You should see a similar page like below if you have done the whole thing right.

![](assets/img/ym-options-demo-plugin-demo-page-1024x579.png)

Great if you have done everything right and see the page above.

Well, maybe some of you, new to plugin development, won't do everything right. For those of you who missed a few steps, the code at this stage should look like below. :-)

<script src="https://gist.github.com/mymizan/0e7263d20ad3234a7a54e2297277cce6.js"></script>

Download the above code from github gists and replace it with everything in the file we mentioned. Things from here, will need a little more focus. As we will be adding markup and some options to our pages, making it a little more complex.

## Register Options

On the settings page we made above, we will be putting a form. WordPress will handle the complete process of updating the form, lessening the burden on us. But WordPress won't save just any form data you put in there, because then the user end will be able to put any data into the database, which isn't a great idea as you have already guessed.

So all of your settings, first, need to be registered by calling _register\_setting_ function. WordPress will only update those form fields if they have been registered already, making it safer for us all.

Add the method below in the class. We have registered three settings - name, age, and gender - all prefixed by ymoptions\_ to prevent option name collision in the database.

```
/** 
 * Register three options
**/
public function register_options()
{
  register_setting( 'ymoptions-group', 'ymoptions_name' );
  register_setting( 'ymoptions-group', 'ymoptions_age' );
  register_setting( 'ymoptions-group', 'ymoptions_gender' );
}
```

Now we need to define another hook in the constructor, that will call this method to register the settings. We will run it only on the admin backend, so we added the is\_admin() check.

```
if ( is_admin() )
{
   add_action( 'admin_init', array($this, 'register_options') );
}
```

## Final Stage: Form Markup

Finally, we will write the markup for the form. Update the method _menu\_markup()_ with the following code.

<script src="https://gist.github.com/mymizan/ef34d7f3574f8aaf6e19835d5b60bc86.js"></script>

What's the meaning of the mess?

The HTML follows certain structure to match the design of WordPress option pages. It's basically a copy-paste, including the form opening tag. But pay attention to the settings\_fields() and submit\_button() functions. The first one takes the option group name that we defined in register\_setting() as a parameter and sets up the necessary hidden field, such a nonce, to let WordPress do its magic and the second one adds the submit button.

Save your changes and go to the options page. You should see a page like below.

![](assets/img/ym-options-completed-demo-plugin-1024x578.png)

## Completed Code

In case you missed any steps or want to see the whole code at once for your reference, take a look at the gist below.

<script src="https://gist.github.com/mymizan/55f79ff9a0689c171fcb0b4cab8427b1.js"></script>

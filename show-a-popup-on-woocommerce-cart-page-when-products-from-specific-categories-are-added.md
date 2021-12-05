---
layout: default
title: "Show a Popup on WooCommerce Cart Page When Products from Specific Categories are Added"
date: "2019-05-29"
categories: 
  - "wordpress"
---

In this tutorial, I will show you how to show a nice popup when user cart contains products from specific categories.

Let's start by defining two constants that will hold some values for us that can easily be changed whenever needed.

## Define Necessary Constants

```
/**
**
** I like to prefix my functions, constants and 
** variables in global scope 
** with YMCODE_/ymcode_ to prevent collision. 
** Feel free to change it. 
**/

/**
** Set your custom message. Do not include html 
** as it will be stripped off. 
**/ 
define('YMCODE_CUSTOM_MESSAGE', 'You have a special discount');

/**
** Set one or more categories IDs in the array. 
**/ 
define('YMCODE_CAT_IDS', [1,2, ]);
```

## Function to show JavaScript Alert

I know I said popups. But I will leave it to you as that will make the code unnecessarily large. The goal here is to show the WordPress functions. However, it's trivial to switch the JavaScript alert to a pop-up library.

Lets write the function that will echo the message to the users.

```
/**
** Add the popup code in JavaScript
**/ 
function ymcode_woocommerce_popup_alert()
{
?>
<script type="text/javascript">
    (function($) {
	alert('<?php echo esc_html(YMCODE_CUSTOM_MESSAGE); ?>');
     })( jQuery );
</script>
<?php
}
```

## Function to check if the cart has products from given categories

The function below will run via a hook when users visit the cart page. It will check if the products in the cart are from specific categories and if so, will hook the function to show the popup.

```
/**
** Check if cart has products from the categories mentioned above
**/
function ymcode_woocommerce_check_if_product_exists()
{
	global $woocommerce;
    $items = $woocommerce->cart->get_cart();

    $show_popup_alert = false; 

    foreach($items as $item => $values) { 
        $terms = get_the_terms( $values['data']->get_id(), 'product_cat' );
		foreach ($terms as $term) {
		    if (in_array($term->term_id, YMCODE_CAT_IDS))
		    {
		    	$show_popup_alert = true;
		    	break 2;
		    }
		}
    }

    if ( $show_popup_alert === true )
    {
    	//product exists, hook the javascript alert message to footer
	add_action( 'wp_footer', 'ymcode_woocommerce_popup_alert' );
    } 

}
```

## Hook the function to check for product categories

Almost done! We just need to hook the above function to run on cart page.

```
add_action('woocommerce_before_cart','ymcode_woocommerce_check_if_product_exists');
```

## Complete Code

Download the complete, formatted code from github gists.

<script src="https://gist.github.com/mymizan/eccd89985ab0948d974363da9014882d.js"></script>

## How to use the code

Well, in case, you are not a developer and wondering where to copy the code, download the complete code from github gist, embedded above. Then change the contestants at the top and place them at the end of your active theme's functions.php

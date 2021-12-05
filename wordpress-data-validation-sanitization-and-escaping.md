---
layout: default
title: "WordPress Data Validation, Sanitization and Escaping"
date: "2019-06-29"
categories: 
  - "wordpress"
---

Coding defensively is very important because your users are free to enter whatever data they want, and some of them will do so with malicious intent. As a programmer, you must ensure that you are validating, and sanitizing user input when saving into the database and also properly escaping when displaying the data back to the user, even if the data is coming from the database and assumed safe.

WordPress has a few functions to help you deal with data validation, sanitization and escaping. We will learn them in this article.

## Data Validation

Data validation means to check that the user has entered the data you requested. If you asked for integer, it means checking that the user has indeed provided a valid integer value.

To validate, you are mostly going to use different PHP functions.

```
//check if the string entered matches the length requested
if ( strlen($user_input) == 5 )
{
  //user entered 5 digits. Proceed to the next step
}

//check if user entered integer
if ( is_int($user_input) )
{
  //the entered value is integer
}

//check if entered data is a numeric value. The difference between is_int and is_numeric is that the later one will also accept floats

if ( is_numeric($user_input) )
{
  //the entered value is numeric. 
}

//check if value is an array 
if ( is_array($user_input) )
{
  //the input is an array
}

//validate an email address
if ( filter_var($user_input, FILTER_VALIDATE_EMAIL) )
{
  //user entered an email address. Save to proceed. 
}

//validate a valid URL
if ( filter_var($user_input, FILTER_VALIDATE_URL) )
{
  //user entered valid URL 
}
```

### Casting into a Desired Type

Alternatively or in addition to validating the user input, and depending on the situation, you may also cast it to a specific type.

```
//cast to integer
$desired_value = (int) $user_input; 

//cast to float
$desired_value = (float) $user_input; 

//though less useful, you can also cast into arrays
$desired_value = (array) $user_input; 
```

## Sanitization in WordPress

The filter\_var() function that we have seen in the above example also has filters to sanitize data. You may check the PHP [documentation](https://www.php.net/manual/en/function.filter-var.php) for details.

In this section, we will check the data sanitization functions provided by WordPress.

```
// Makes the data safe for html input fields of type text
sanitize_text_field( $input_string )

// Makes the data safe for html input fields of type text
sanitize_textarea_field( $input_string )

//strip all html tags
wp_strip_all_tags( $input_string )

// Strip tags using PHP function (will not eliminate 
// the contents of script and style tags, unlike wp_strip_all_tags)
strip_tags( $string )

//Remove all characters that are not allowed in an email address
sanitize_email( $email )

//Makes the string save to be used in SQL query
esc_sql( $sql )
```

## Escaping

One important note to remember is that you should always escape data, even if it's coming from the database. You should never assume that data you are going to display is safe in any way because, who knows, your database may as well be compromised.

So when displaying data, always use the functions below to escape them properly.

```
//Escape html for display
<p> <?php echo esc_html( $text ) ?> </p>

//Escape URL
<a href='<?php echo esc_url( $entered_url ); ?>'> TEST LINK </a>

//Escape the input for text area
esc_textarea( $text );
```

You will find the the list of all the available WordPress sanitization, validation, and escaping functions in /wp-includes/formatting.php file.

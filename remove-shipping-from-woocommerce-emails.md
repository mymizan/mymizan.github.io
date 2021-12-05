---
layout: default
title: "Remove Shipping Row From WooCommerce Emails"
date: "2019-05-24"
categories: 
  - "life"
---

If you have free shipping for all your products in your store and you have already shown that to your customers, you may want to remove the Free Shipping row from the email sent to customers after purchase.

In this article, I will show you three ways to remove the free shipping row from email sent to customers.

## Remove Shipping Row With Filter

Calling this filter will remove shipping method form the page, email and invoice mailed to customers.

Add this code in your plugin file or your active theme's functions.php

```
/**
** Remove the shipping row from woocommerce_get_order_item_totals array 
**/ 
function ymcode_remove_shipping_details_from_woocommerce_email($total_rows, $obj, $tax_display)
{
	if ( isset($total_rows['shipping']) )
	{
		unset($total_rows['shipping']);
	}

	return $total_rows;
}

add_filter( 'woocommerce_get_order_item_totals', 'ymcode_remove_shipping_details_from_woocommerce_email', 10, 3 );
```

## Hide Shipping Row with CSS

```
/**
** Shipping is 2nd row, hiding it with CSS
**/
function ymcode_hide_shipping_row_with_css()
{
	?>
	<style>
          tfoot tr:nth-child(2) {
             display: none !important;
	  }
	</style>
	<?php
}

add_action('woocommerce_email_header', 'ymcode_hide_shipping_row_with_css', 10, 0);
```

## Remove Shipping row by editing the template file

Place the below code in a file under your active theme's folder/woocommerce/emails/email-order-details.php

```
<?php
/**
 * Order details table shown in emails.
 *
 * This template can be overridden by copying it to yourtheme/woocommerce/emails/email-order-details.php.
 *
 * HOWEVER, on occasion WooCommerce will need to update template files and you
 * (the theme developer) will need to copy the new files to your theme to
 * maintain compatibility. We try to do this as little as possible, but it does
 * happen. When this occurs the version of the template file will be bumped and
 * the readme will list any important changes.
 *
 * @see https://docs.woocommerce.com/document/template-structure/
 * @package WooCommerce/Templates/Emails
 * @version 3.3.1
 */

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

$text_align = is_rtl() ? 'right' : 'left';

do_action( 'woocommerce_email_before_order_table', $order, $sent_to_admin, $plain_text, $email ); ?>

<h2>
	<?php
	if ( $sent_to_admin ) {
		$before = '<a class="link" href="' . esc_url( $order->get_edit_order_url() ) . '">';
		$after  = '</a>';
	} else {
		$before = '';
		$after  = '';
	}
	/* translators: %s: Order ID. */
	echo wp_kses_post( $before . sprintf( __( '[Order #%s]', 'woocommerce' ) . $after . ' (<time datetime="%s">%s</time>)', $order->get_order_number(), $order->get_date_created()->format( 'c' ), wc_format_datetime( $order->get_date_created() ) ) );
	?>
</h2>

<div style="margin-bottom: 40px;">
	<table class="td" cellspacing="0" cellpadding="6" style="width: 100%; font-family: 'Helvetica Neue', Helvetica, Roboto, Arial, sans-serif;" border="1">
		<thead>
			<tr>
				<th class="td" scope="col" style="text-align:<?php echo esc_attr( $text_align ); ?>;"><?php esc_html_e( 'Product', 'woocommerce' ); ?></th>
				<th class="td" scope="col" style="text-align:<?php echo esc_attr( $text_align ); ?>;"><?php esc_html_e( 'Quantity', 'woocommerce' ); ?></th>
				<th class="td" scope="col" style="text-align:<?php echo esc_attr( $text_align ); ?>;"><?php esc_html_e( 'Price', 'woocommerce' ); ?></th>
			</tr>
		</thead>
		<tbody>
			<?php
			echo wc_get_email_order_items( $order, array( // WPCS: XSS ok.
				'show_sku'      => $sent_to_admin,
				'show_image'    => false,
				'image_size'    => array( 32, 32 ),
				'plain_text'    => $plain_text,
				'sent_to_admin' => $sent_to_admin,
			) );
			?>
		</tbody>
		<tfoot>
			<?php
			$totals = $order->get_order_item_totals();

			if ( $totals ) {
				$i = 0;
				foreach ( $totals as $total ) {
					$i++;

					if ($i == 2) 
						continue; //shipping row is the second row in array
					?>
					<tr>
						<th class="td" scope="row" colspan="2" style="text-align:<?php echo esc_attr( $text_align ); ?>; <?php echo ( 1 === $i ) ? 'border-top-width: 4px;' : ''; ?>"><?php echo wp_kses_post( $total['label'] ); ?></th>
						<td class="td" style="text-align:<?php echo esc_attr( $text_align ); ?>; <?php echo ( 1 === $i ) ? 'border-top-width: 4px;' : ''; ?>"><?php echo wp_kses_post( $total['value'] ); ?></td>
					</tr>
					<?php
				}
			}
			if ( $order->get_customer_note() ) {
				?>
				<tr>
					<th class="td" scope="row" colspan="2" style="text-align:<?php echo esc_attr( $text_align ); ?>;"><?php esc_html_e( 'Note:', 'woocommerce' ); ?></th>
					<td class="td" style="text-align:<?php echo esc_attr( $text_align ); ?>;"><?php echo wp_kses_post( wptexturize( $order->get_customer_note() ) ); ?></td>
				</tr>
				<?php
			}
			?>
		</tfoot>
	</table>
</div>

<?php do_action( 'woocommerce_email_after_order_table', $order, $sent_to_admin, $plain_text, $email ); ?>
```

You can download the code from [Github gists](https://gist.github.com/mymizan/a86781053b53c1bd25bbca3f4d2f24b1) too.

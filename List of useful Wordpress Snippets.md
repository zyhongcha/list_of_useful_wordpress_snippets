# Handy Wordpress Snippets for Developers (WIP)
These are some of wordpress optimizations/ adjustments that I have encountered on my work. Use accordingly to your situation.


## Modify Wordpress in functions.php


### Remove emojis that are loaded on default 
remove_action( 'wp_head', 'print_emoji_detection_script', 7 );
remove_action( 'wp_print_styles', 'print_emoji_styles' );

### Deactivate xmlrpc.php for security reasons (note: server-side configuration is also recommended)
```
remove_action('wp_head', 'rsd_link');
add_filter( 'xmlrpc_enabled', '__return_false' );
```
#### Disable on server as well:
Apache Server on .htaccess file:
```
<Files xmlrpc.php>
Order Deny,Allow
Deny from all
</Files>
```
Nginx Server on nginx.conf:
```
server {
    location = /xmlrpc.php {
        deny all;
    }
}
```


### Modify all links in Wordpress to be relative instead of absolute, very useful for multilang sites
```
add_action( 'template_redirect', 'rw_relative_urls' );
function rw_relative_urls() {
    // Don't do anything if:
    // - In feed
    // - In sitemap by WordPress SEO plugin
    if ( is_feed() || get_query_var( 'sitemap' ) )
        return;
    $filters = array(
        'post_link',
        'post_type_link',
        'page_link',
        'attachment_link',
        'get_shortlink',
        'post_type_archive_link',
        'get_pagenum_link',
        'get_comments_pagenum_link',
        'term_link',
        'search_link',
        'day_link',
        'month_link',
        'year_link',
		'author_link',
    );
    foreach ( $filters as $filter )
    {
        add_filter( $filter, 'wp_make_link_relative' );
    }
}
```

### Disable default "website" input field in commentary
```
add_filter('comment_form_default_fields', 'website_remove');
function website_remove($fields)
{
if(isset($fields['url']))
unset($fields['url']);
return $fields;
}
```


### Add language hreftags in HTML head for multilang sites (do not forget to add canonical link as well if you use this)
```
// Add Language Tags depending on current domain
add_action('wp_head', 'addHrefLang');
function addHrefLang(){
?><link rel="alternate" hreflang="de-AT" href="https://www.at.example.de/" title="de_AT"/>
<link rel="alternate" hreflang="de-CH" href="https://www.ch.example.de/" title="de_CH"/>
// Original (canonical link) page:
<link rel="alternate" hreflang="de-DE" href="https://www.example.de/" title="de_DE"/>
}
```


### Disable pingbacks
```
function no_self_ping( &$links ) {
    $home = get_option( 'home' );
    foreach ( $links as $l => $link )
        if ( 0 === strpos( $link, $home ) )
            unset($links[$l]);
}
 
add_action( 'pre_ping', 'no_self_ping' );
```

## Modify Wordpress using PHP

### Reset comment count in a single post after having them deleted manually in database. This code loops through all posts.
```
$entries = $wpdb->get_results("SELECT * FROM wp_posts WHERE post_type IN ('post', 'page')");

foreach($entries as $entry)
{
    $post_id = $entry->ID;
    $comment_count = $wpdb->get_var("SELECT COUNT(*) AS comment_cnt FROM wp_comments WHERE comment_post_ID = '$post_id' AND comment_approved = '1'");
    $wpdb->query("UPDATE wp_posts SET comment_count = '$comment_count' WHERE ID = '$post_id'");
}
```

```
/* add custom html header field to admin panel*/

add_filter('admin_init', 'my_general_settings_register_fields');

function my_general_settings_register_fields()
{
	register_setting('general', 'custom_html', 'esc_attr');
	add_settings_field('custom_html', '<label for="custom_html">' . __('Scripte im HTML-Head', 'custom_html') . '</label>', 'my_general_custom_html', 'general');
}

function my_general_custom_html()
{
	$custom_html = get_option('custom_html', '');
	//echo '<input id="custom_html" style="width: 35%;" type="text" name="custom_html" value="' . $custom_html . '" />';
	echo '<textarea name="custom_html" rows="10" cols="50" id="custom_html" class="large-text code">' . $custom_html . '</textarea>';
}


// Inserting a script in the WordPress head

function my_google_header_code()
{
	echo html_entity_decode(get_option('custom_html'), ENT_QUOTES);
}

add_action('wp_head', 'my_google_header_code');

// END inserting script in WP head


/* END add custom html header field to admin panel*/




//Make all links in Wordpress relative - CRUCIAL FOR Multi-Language Subdomains (at., de., ch.,) to show correct links
add_action( 'template_redirect', 'rw_relative_urls' );
function rw_relative_urls() {
    // Don't do anything if:
    // - In feed
    // - In sitemap by WordPress SEO plugin
    if ( is_feed() || get_query_var( 'sitemap' ) )
        return;
    $filters = array(
        'post_link',
        'post_type_link',
        'page_link',
        'attachment_link',
        'get_shortlink',
        'post_type_archive_link',
        'get_pagenum_link',
        'get_comments_pagenum_link',
        'term_link',
        'search_link',
        'day_link',
        'month_link',
        'year_link',
		'author_link',
    );
    foreach ( $filters as $filter )
    {
        add_filter( $filter, 'wp_make_link_relative' );
    }
}


// Set cookie on page (lifespan 1 Week / 604800 secs)
function pageVisitedCookie(){

  if (empty($_SERVER["QUERY_STRING"])&& !isset($_COOKIE["COOKIE_NAME_HERE"])) {
	  header('set-cookie: '.rawurlencode('COOKIE_NAME_HERE').'='.rawurlencode('').'; max-age=604800', false);
  }
}
// add_action('init', 'pageVisitedCookie');


```




## Optional Styles

### Add custom styles to wp-login.php page
```
function customStylesLogin() { ?>
    <style type="text/css">
    /* This img replaces the default wordpress logo */
        #login h1 a, .login h1 a {
            background-image: url("/wp-content/uploads/2020/07/MY_CUSTOM_LOGO.png");
		width:320px;
		background-position: center;
		mix-blend-mode: multiply;
		background-size: 320px;
		background-repeat: no-repeat;
        	padding-bottom: 30px;
        }
		body {
			background: #e4e8ea!important;
		}

		.wp-core-ui .button-primary {
			background: #008db7!important;
			border-color: #008db7!important;
		}
    </style>
<?php }
add_action( 'login_enqueue_scripts', 'customStylesLogin' );
```


## SQL Database Snippets
### Delete all post revisions (not just in wp_posts)
```
DELETE a,b,c FROM wp_posts a LEFT JOIN wp_term_relationships b ON (a.ID = b.object_id)
LEFT JOIN wp_postmeta c ON (a.ID = c.post_id) WHERE a.post_type = 'revision'
```

### Delete orphaned posts in wp_postmeta
```
DELETE pm
FROM wp_postmeta pm
LEFT JOIN wp_posts wp ON wp.ID = pm.post_id
WHERE wp.ID IS NULL
```

### Delete all trashed posts 
```
DELETE p
FROM wp_posts p
LEFT OUTER JOIN wp_postmeta pm ON (p.ID = pm.post_id)
WHERE post_status = 'trash'
```



## wp_Config Additions
```
// Deactivate Post Revisions
define('WP_POST_REVISIONS', false);

// Save only max. 3 post revisions
define( ‘WP_POST_REVISIONS’, 3 );

// Automatically delete trashed posts after 30 days (set to 0 for disabling trashed posts)
define( 'EMPTY_TRASH_DAYS', 30 ); 
```



## WooCommerce additional settings


###
```
//Disable logging Action Scheduler, REDUCES DB BLOAT 
add_filter( 'action_scheduler_retention_period', '__return_zero' );


//Disable standard Wordpress auto-generated image sizes
add_filter('intermediate_image_sizes_advanced', 'disable_wp_responsive_image_sizes');
function disable_wp_responsive_image_sizes($sizes) {

	unset($sizes['large']);
	unset($sizes['medium_large']);
	return $sizes;
}

// Override WooCommerce Thumbnails to 150x150 instead default 100x100px
add_filter( 'woocommerce_get_image_size_gallery_thumbnail', function( $size ) {
	return array(
		'width'  => 150,
		'height' => 150,
		'crop'   => 1,
	);
} );


// Disable pre selecting "ship to different address".
add_filter( 'woocommerce_ship_to_different_address_checked', '__return_false' );

//Disable WooCommerce Bloat
add_filter( 'woocommerce_admin_disabled', '__return_true' );
add_filter( 'woocommerce_marketing_menu_items', '__return_empty_array' );
add_filter( 'woocommerce_helper_suppress_admin_notices', '__return_true' );


//Remove "What is paypal" image
add_filter('woocommerce_gateway_icon', 'remove_what_is_paypal', 10, 2);
function remove_what_is_paypal($icon_html, $gateway_id)
{
	if ('paypal' == $gateway_id) {
		$icon_html = '<img src="/wp-content/plugins/woocommerce/includes/gateways/paypal/assets/images/paypal.png" alt="PayPal Acceptance Mark">';
	}
	return $icon_html;
}



// Add feature: Link above change product price field to apply the change to all its variations.
add_action( 'woocommerce_product_data_panels', 'gowp_global_variation_price' );  
function gowp_global_variation_price() {  	
	global $woocommerce;  	?>  		
	<script type="text/javascript">  			
	function addVariationLinks() {  				
		a = jQuery( '<a href="#">Apply change to all variations</a>' );  				
		b = jQuery( 'input[name^="variable_regular_price"].wc_input_price' );  				
		a.click( function( c ) {  					
			d = jQuery( this ).parent( 'label' ).next( 'input[name^="variable_regular_price"].wc_input_price' ).val();
			e = confirm( "Change price to all variations " + d + "?" );  					
			if ( e ) b.val( d ).trigger( 'change' );  					
			c.preventDefault();  				} );  				
			b.prev( 'label' ).append( " " ).append( a );  				
			aa = jQuery( '<a href="#">Auf alle Variationen übertragen</a>' );  				
			bb = jQuery( 'input[name^="variable_sale_price"].wc_input_price' );  				
			aa.click( function( cc ) {  					
				dd = jQuery( this ).parent( 'label' ).next( 'input[name^="variable_sale_price"].wc_input_price' ).val();
				ee = confirm( "Change price to all variations " + dd + "?" );  					
				if ( ee ) bb.val( dd ).trigger( 'change' );  					
				cc.preventDefault();  				} );  				
				bb.prev( 'label' ).append( " " ).append( aa );  			}  			
				<?php if ( version_compare( $woocommerce->version, '2.4', '>=' ) ) : ?>  				
				jQuery( document ).ready( function() {  					
					jQuery( document ).ajaxComplete( function( event, request, settings ) {  						
						if ( settings.data.lastIndexOf( "action=woocommerce_load_variations", 0 ) === 0 ) {
						addVariationLinks();  						}  					} );  				} );  			
						<?php else: ?>  				
						addVariationLinks();  			
						<?php endif; ?>  		</script>
						  	<?php  }






// Modify WooCommerce Related Product function by displaying only products from same specific attribute and category //ex. attribute: pa_marke
function woocommerce_aantal_related( $args ) {
	$args['posts_per_page'] = 3;
	$args['columns'] = 3;
	return $args;
}
add_filter( 'woocommerce_output_related_products_args', 'woocommerce_aantal_related' );

function filter_related_products($args){	
	global $product;
	$args = array();	
	$thiscats = wc_get_product_terms( $product->id, 'product_cat', array( 'fields' => 'slugs' ) );
	$thistypes = wc_get_product_terms( $product->id, 'pa_marke', array( 'fields' => 'slugs' ) );
	
	$ids_by_model_attribute = get_posts( array(
        'post_type' => 'product',
        'numberposts' => -1,
        'post_status' => 'publish',
        'fields' => 'ids',
		'post__not_in' => array( $product->get_id() ), // exclude current product
		'tax_query' => array(
			'relation' => 'AND',
				array(
					'taxonomy' => 'product_cat',
					'field'    => 'slug',
					'terms'    => $cats,
					'operator' => 'AND'
				),
			array(
				'taxonomy' => 'pa_marke',
				'field'    => 'slug',
				'terms'    => $thistypes,
				'operator' => 'AND'
			)
		)
   ) );
	
	if($ids_by_model_attribute){
		foreach($ids_by_model_attribute as $product_id){
			$cats = wc_get_product_terms( $product_id, 'product_cat', array( 'fields' => 'slugs' ) );
			$types = wc_get_product_terms( $product_id, 'pa_marke', array( 'fields' => 'slugs' ) );
			if($cats == $thiscats && $thistypes == $types){
				array_push($args,$product_id);
			}
		}
	}
	
	return $args;
}
add_filter('woocommerce_related_products','filter_related_products');




/**
 * Hide shipping rates when free shipping is available.
 * Updated to support WooCommerce 2.6 Shipping Zones.
 *
 * @param array $rates Array of rates found for the package.
 * @return array
 */
function my_hide_shipping_when_free_is_available( $rates ) {
	$free = array();

	foreach ( $rates as $rate_id => $rate ) {
		if ( 'free_shipping' === $rate->method_id ) {
			$free[ $rate_id ] = $rate;
			break;
		}
	}

	return ! empty( $free ) ? $free : $rates;
}


// Change "SALE" Label on Product Gallery Page
add_filter('woocommerce_sale_flash', 'woocommerce_custom_sale_text', 10, 3);
function woocommerce_custom_sale_text($text, $post, $_product)
{
    return '<span class="onsale">SALE</span>';
}


```
###

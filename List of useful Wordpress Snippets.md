# Handy Wordpress Snippets for Developers (WIP)



## Modify Wordpress in functions.php


### Remove emojis that are loaded on default 
remove_action( 'wp_head', 'print_emoji_detection_script', 7 );
remove_action( 'wp_print_styles', 'print_emoji_styles' );

### Deactivate xmlrpc.php for security reasons (note: server-side configuration is also recommended)
```
remove_action('wp_head', 'rsd_link');
add_filter( 'xmlrpc_enabled', '__return_false' );
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

// Automatically delete trashed posts after 30 days
define( 'EMPTY_TRASH_DAYS', 30 ); 
```
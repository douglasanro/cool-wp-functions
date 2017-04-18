# Cool WP Functions

> A collection of cool WordPress functions.

* [Remove WordPress version from header](#remove-wordpress-version-from-header)
* [Disable built-in post type](#disable-built-in-post-type)
* [Disable WordPress emoji](#disable-built-in-post-type)
* [Remove comments](#remove-comments)
* [Add CPT to your main WP RSS feed](#add-cpt-to-your-main-wp-rss-feed)
* [Add CPT count action to WordPress dashboard](#add-cpt-count-action-to-wordpress-dashboard)
* [Create custom WordPress dashboard widget](#create-custom-wordpress-dashboard-widget)
* [Add breadcrumbs to your theme](#add-breadcrumbs-to-your-theme)
* [Add numeric pagination to your theme](#add-numeric-pagination-to-your-theme)
* [Fix custom post type tags pagination](#fix-custom-post-type-tags-pagination)
* [Remove default image sizes](#remove-default-image-sizes)
* [Add CPT to category and tag archives](#add-cpt-to-category-and-tag-archives)
* [Add the support for your CPT in the widget activity of the admin dashboard](#add-the-support-for-your-cpt-in-the-widget-activity-of-the-admin-dashboard)
* [Remove all dashboard widgets](#remove-all-dashboard-widgets)
* [Remove welcome panel in dashboard](#remove-welcome-panel-in-dashboard)
* [Let WordPress manage the document title](#let-wordpress-manage-the-document-title)
* [Switch default core markup to output valid HTML5](#switch-default-core-markup-to-output-valid-html5)
* [Modify admin footer text](#modify-admin-footer-text)
* [Modify excerpt length](#modify-excerpt-length)
* [Change more excerpt](#change-more-excerpt)
* [Create WordPress settings page For custom options](#create-wordpress-settings-page-for-custom-options)
* [Add Bootstrap 3 navigation style to WordPress menu manager](#add-bootstrap-3-navigation-style-to-wordpress-menu-manager)

## Remove WordPress version from header

```php
/* ----------------------------------------------------------------------------
 * Remove WordPress version from header
 * ------------------------------------------------------------------------- */

function simple_remove_version_info() {
	return '';
}
add_filter( 'the_generator', 'simple_remove_version_info' );
```

## Disable built-in post type

Use this to remove `Posts` if your theme dont use it at all.

```php
/* ----------------------------------------------------------------------------
 * Disable built-in post type
 * ------------------------------------------------------------------------- */

add_action( 'init', 'simple_remove_default_post_type', 1 );
function simple_remove_default_post_type() {
		register_post_type( 'post', array(
				'map_meta_cap' => false
		) );
}
```

## Disable WordPress emoji

```php
/* ----------------------------------------------------------------------------
 * Disable WordPress emoji
 * ------------------------------------------------------------------------- */

function disable_wp_emojicons() {
	remove_action( 'admin_print_styles', 'print_emoji_styles' );
	remove_action( 'wp_head', 'print_emoji_detection_script', 7 );
	remove_action( 'admin_print_scripts', 'print_emoji_detection_script' );
	remove_action( 'wp_print_styles', 'print_emoji_styles' );
	remove_filter( 'wp_mail', 'wp_staticize_emoji_for_email' );
	remove_filter( 'the_content_feed', 'wp_staticize_emoji' );
	remove_filter( 'comment_text_rss', 'wp_staticize_emoji' );
	add_filter( 'tiny_mce_plugins', 'disable_emojicons_tinymce' );
	add_filter( 'emoji_svg_url', '__return_false' );
}
add_action( 'init', 'disable_wp_emojicons' );
function disable_emojicons_tinymce( $plugins ) {
	if ( is_array( $plugins ) ) {
		return array_diff( $plugins, array( 'wpemoji' ) );
	} else {
		return array();
	}
}
```

## Remove comments

```php
/* ----------------------------------------------------------------------------
 * Remove comments
 * ------------------------------------------------------------------------- */

// Removes from admin menu
add_action( 'admin_menu', 'my_remove_admin_menus' );
function my_remove_admin_menus() {
	remove_menu_page( 'edit-comments.php' );
}
// Removes from post and pages
add_action( 'init', 'remove_comment_support', 100 );
function remove_comment_support() {
	remove_post_type_support( 'post', 'comments' );
	remove_post_type_support( 'page', 'comments' );
}
// Removes from admin bar
function mytheme_admin_bar_render() {
	global $wp_admin_bar;
	$wp_admin_bar->remove_menu( 'comments' );
}
add_action( 'wp_before_admin_bar_render', 'mytheme_admin_bar_render' );
```

## Add CPT to your main WP RSS feed

```php
/* ----------------------------------------------------------------------------
 * Add CPT to your main WordPress RSS feed
 * ------------------------------------------------------------------------- */

function simple_feed_request( $rss ) {
	if ( isset( $rss[ 'feed' ] ) && !isset( $rss['post_type'] ) ) {
		// Return all post types
		$post_types = get_post_types();
		$rss['post_type'] = $post_types;
		// Return posts of post types of your choice like 'post' and 'news'
		//$rss['post_type'] = array( 'post', 'news' );
	}
	return $rss;
}
```

## Add CPT count action to WordPress dashboard

```php
/* ----------------------------------------------------------------------------
 * Add CPT count action to WordPress dashboard
 * ------------------------------------------------------------------------- */

add_action( 'dashboard_glance_items', 'simple_glance_items' );

// Showing all custom posts count
function simple_glance_items() {
	$glances = array();

	$args = array(
		'public'   => true, // Showing public post types only
		'_builtin' => false // Except the build-in wp post types (page, post, attachments)
	);

	// Getting your custom post types
	$post_types = get_post_types( $args, 'object', 'and' );

	foreach ( $post_types as $post_type ) {
		// Counting each post
		$num_posts = wp_count_posts( $post_type->name );

		// Number format
		$num   = number_format_i18n( $num_posts->publish );
		// Text format
		$text  = _n( $post_type->labels->singular_name, $post_type->labels->name, intval( $num_posts->publish ) );

		// If use capable to edit the post type
		if ( current_user_can( 'edit_posts' ) ) {
				// Show with link
				$glance = '<a class="'.$post_type->name.'-count" href="'.admin_url( 'edit.php?post_type='.$post_type->name ).'">'.$num.' '.$text.'</a>';
		} else {
			// Show without link
			$glance = '<span class="'.$post_type->name.'-count">'.$num.' '.$text.'</span>';
		}

		// Save in array
		$glances[] = $glance;
	}

	// return them
	return $glances;
}
```

## Create custom WordPress dashboard widget

```php
/* ----------------------------------------------------------------------------
 * Create custom WordPress dashboard widget
 * ------------------------------------------------------------------------- */

function dashboard_widget_function() {
	echo '
		<h2>Custom Dashboard Widget</h2>
		<p>Custom content here</p>
	';
}

function add_dashboard_widgets() {
	wp_add_dashboard_widget( 'custom_dashboard_widget', 'Custom Dashoard Widget', 'dashboard_widget_function' );
}
add_action( 'wp_dashboard_setup', 'add_dashboard_widgets' );
```

## Add breadcrumbs to your theme

```php
/* ----------------------------------------------------------------------------
 * Add breadcrumbs to your theme
 * ------------------------------------------------------------------------- */

function simple_breadcrumbs() {

	/* === OPTIONS === */
	$text['home']     = 'Home'; // text for the 'Home' link
	$text['category'] = 'Archive by Category "%s"'; // text for a category page
	$text['search']   = 'Search Results for "%s" Query'; // text for a search results page
	$text['tag']      = 'Posts Tagged "%s"'; // text for a tag page
	$text['author']   = 'Articles Posted by %s'; // text for an author page
	$text['404']      = 'Error 404'; // text for the 404 page

	$show_current   = 1; // 1 - show current post/page/category title in breadcrumbs, 0 - don't show
	$show_on_home   = 0; // 1 - show breadcrumbs on the homepage, 0 - don't show
	$show_home_link = 1; // 1 - show the 'Home' link, 0 - don't show
	$show_title     = 1; // 1 - show the title for the links, 0 - don't show
	$delimiter      = ' &raquo; '; // delimiter between crumbs
	$before         = '<span class="current">'; // tag before the current crumb
	$after          = '</span>'; // tag after the current crumb
	/* === END OF OPTIONS === */

	global $post;
	$home_link    = home_url('/');
	$link_before  = '<span typeof="v:Breadcrumb">';
	$link_after   = '</span>';
	$link_attr    = ' rel="v:url" property="v:title"';
	$link         = $link_before . '<a' . $link_attr . ' href="%1$s">%2$s</a>' . $link_after;
	$parent_id    = $parent_id_2 = $post->post_parent;
	$frontpage_id = get_option('page_on_front');

	if (is_home() || is_front_page()) {

		if ($show_on_home == 1) echo '<div class="breadcrumbs"><a href="' . $home_link . '">' . $text['home'] . '</a></div>';

	} else {

		echo '<div class="breadcrumbs" xmlns:v="http://rdf.data-vocabulary.org/#">';
		if ($show_home_link == 1) {
			echo '<a href="' . $home_link . '" rel="v:url" property="v:title">' . $text['home'] . '</a>';
			if ($frontpage_id == 0 || $parent_id != $frontpage_id) echo $delimiter;
		}

		if ( is_category() ) {
			$this_cat = get_category(get_query_var('cat'), false);
			if ($this_cat->parent != 0) {
				$cats = get_category_parents($this_cat->parent, TRUE, $delimiter);
				if ($show_current == 0) $cats = preg_replace("#^(.+)$delimiter$#", "$1", $cats);
				$cats = str_replace('<a', $link_before . '<a' . $link_attr, $cats);
				$cats = str_replace('</a>', '</a>' . $link_after, $cats);
				if ($show_title == 0) $cats = preg_replace('/ title="(.*?)"/', '', $cats);
				echo $cats;
			}
			if ($show_current == 1) echo $before . sprintf($text['category'], single_cat_title('', false)) . $after;

		} elseif ( is_search() ) {
			echo $before . sprintf($text['search'], get_search_query()) . $after;

		} elseif ( is_day() ) {
			echo sprintf($link, get_year_link(get_the_time('Y')), get_the_time('Y')) . $delimiter;
			echo sprintf($link, get_month_link(get_the_time('Y'),get_the_time('m')), get_the_time('F')) . $delimiter;
			echo $before . get_the_time('d') . $after;

		} elseif ( is_month() ) {
			echo sprintf($link, get_year_link(get_the_time('Y')), get_the_time('Y')) . $delimiter;
			echo $before . get_the_time('F') . $after;

		} elseif ( is_year() ) {
			echo $before . get_the_time('Y') . $after;

		} elseif ( is_single() && !is_attachment() ) {
			if ( get_post_type() != 'post' ) {
				$post_type = get_post_type_object(get_post_type());
				$slug = $post_type->rewrite;
				printf($link, $home_link . '/' . $slug['slug'] . '/', $post_type->labels->singular_name);
				if ($show_current == 1) echo $delimiter . $before . get_the_title() . $after;
			} else {
				$cat = get_the_category(); $cat = $cat[0];
				$cats = get_category_parents($cat, TRUE, $delimiter);
				if ($show_current == 0) $cats = preg_replace("#^(.+)$delimiter$#", "$1", $cats);
				$cats = str_replace('<a', $link_before . '<a' . $link_attr, $cats);
				$cats = str_replace('</a>', '</a>' . $link_after, $cats);
				if ($show_title == 0) $cats = preg_replace('/ title="(.*?)"/', '', $cats);
				echo $cats;
				if ($show_current == 1) echo $before . get_the_title() . $after;
			}

		} elseif ( !is_single() && !is_page() && get_post_type() != 'post' && !is_404() ) {
			$post_type = get_post_type_object(get_post_type());
			echo $before . $post_type->labels->singular_name . $after;

		} elseif ( is_attachment() ) {
			$parent = get_post($parent_id);
			$cat = get_the_category($parent->ID); $cat = $cat[0];
			$cats = get_category_parents($cat, TRUE, $delimiter);
			$cats = str_replace('<a', $link_before . '<a' . $link_attr, $cats);
			$cats = str_replace('</a>', '</a>' . $link_after, $cats);
			if ($show_title == 0) $cats = preg_replace('/ title="(.*?)"/', '', $cats);
			echo $cats;
			printf($link, get_permalink($parent), $parent->post_title);
			if ($show_current == 1) echo $delimiter . $before . get_the_title() . $after;

		} elseif ( is_page() && !$parent_id ) {
			if ($show_current == 1) echo $before . get_the_title() . $after;

		} elseif ( is_page() && $parent_id ) {
			if ($parent_id != $frontpage_id) {
				$breadcrumbs = array();
				while ($parent_id) {
					$page = get_page($parent_id);
					if ($parent_id != $frontpage_id) {
						$breadcrumbs[] = sprintf($link, get_permalink($page->ID), get_the_title($page->ID));
					}
					$parent_id = $page->post_parent;
				}
				$breadcrumbs = array_reverse($breadcrumbs);
				for ($i = 0; $i < count($breadcrumbs); $i++) {
					echo $breadcrumbs[$i];
					if ($i != count($breadcrumbs)-1) echo $delimiter;
				}
			}
			if ($show_current == 1) {
				if ($show_home_link == 1 || ($parent_id_2 != 0 && $parent_id_2 != $frontpage_id)) echo $delimiter;
				echo $before . get_the_title() . $after;
			}

		} elseif ( is_tag() ) {
			echo $before . sprintf($text['tag'], single_tag_title('', false)) . $after;

		} elseif ( is_author() ) {
			global $author;
			$userdata = get_userdata($author);
			echo $before . sprintf($text['author'], $userdata->display_name) . $after;

		} elseif ( is_404() ) {
			echo $before . $text['404'] . $after;
		}

		if ( get_query_var('paged') ) {
			if ( is_category() || is_day() || is_month() || is_year() || is_search() || is_tag() || is_author() ) echo ' (';
			echo __('Page') . ' ' . get_query_var('paged');
			if ( is_category() || is_day() || is_month() || is_year() || is_search() || is_tag() || is_author() ) echo ')';
		}

		echo '</div><!-- .breadcrumbs -->';

	}
} // end simple_breadcrumbs()
```

Insert this where you want it to appear in your template:

```php
<?php if (function_exists('simple_breadcrumbs')) simple_breadcrumbs(); ?>
```

## Add numeric pagination to your theme

```php
/* ----------------------------------------------------------------------------
 * Add numeric pagination to your theme
 * ------------------------------------------------------------------------- */

function numeric_posts_nav() {

	if( is_singular() )
		return;

	global $wp_query;

	/** Stop execution if there's only 1 page */
	if( $wp_query->max_num_pages <= 1 )
		return;

	$paged = get_query_var( 'paged' ) ? absint( get_query_var( 'paged' ) ) : 1;
	$max   = intval( $wp_query->max_num_pages );

	/**	Add current page to the array */
	if ( $paged >= 1 )
		$links[] = $paged;

	/**	Add the pages around the current page to the array */
	if ( $paged >= 3 ) {
		$links[] = $paged - 1;
		$links[] = $paged - 2;
	}

	if ( ( $paged + 2 ) <= $max ) {
		$links[] = $paged + 2;
		$links[] = $paged + 1;
	}

	echo '<div class="navigation"><ul>' . "\n";

	/**	Previous Post Link */
	if ( get_previous_posts_link() )
		printf( '<li class="arrow">%s</li>' . "\n", get_previous_posts_link('<i class="my-left-arrow"></i>') );

	/**	Link to first page, plus ellipses if necessary */
	if ( ! in_array( 1, $links ) ) {
		$class = 1 == $paged ? ' class="active"' : '';

		printf( '<li%s><a href="%s">%s</a></li>' . "\n", $class, esc_url( get_pagenum_link( 1 ) ), '1' );

		if ( ! in_array( 2, $links ) )
			echo '<li class="points">…</li>';
	}

	/**	Link to current page, plus 2 pages in either direction if necessary */
	sort( $links );
	foreach ( (array) $links as $link ) {
		$class = $paged == $link ? ' class="active"' : '';
		printf( '<li%s><a href="%s">%s</a></li>' . "\n", $class, esc_url( get_pagenum_link( $link ) ), $link );
	}

	/**	Link to last page, plus ellipses if necessary */
	if ( ! in_array( $max, $links ) ) {
		if ( ! in_array( $max - 1, $links ) )
			echo '<li class="points">…</li>' . "\n";

		$class = $paged == $max ? ' class="active"' : '';
		printf( '<li%s><a href="%s">%s</a></li>' . "\n", $class, esc_url( get_pagenum_link( $max ) ), $max );
	}

	/**	Next Post Link */
	if ( get_next_posts_link() )
		printf( '<li class="arrow"/>%s</li>' . "\n", get_next_posts_link('<i class="my-right-arrow"></i>') );

	echo '</ul></div>' . "\n";

 }
```

Insert this where you want it to appear in your template:

```php
<?php if ( function_exists( 'numeric_posts_nav' ) ) { numeric_posts_nav(); } ?>
```

## Fix custom post type tags pagination

If you have `cpt` with `tags` and use `numeric_posts_nav` function add this to fix the pagination.

```php
/* ----------------------------------------------------------------------------
 * Fix custom post type tags pagination
 * ------------------------------------------------------------------------- */

function tagfix_add_custom_types( $query ) {
	if( is_tag() && $query->is_main_query() ) {
		$post_types = get_post_types();
		$query->set( 'post_type', $post_types );
	}
}
add_filter( 'pre_get_posts', 'tagfix_add_custom_types' );
```

## Remove default image sizes

```php
/* ----------------------------------------------------------------------------
 * Remove default image sizes
 * ------------------------------------------------------------------------- */

function remove_default_image_sizes( $sizes ) {

	/* Default WordPress */
	unset( $sizes['thumbnail'] );       // Remove Thumbnail (150 x 150 hard cropped)
	unset( $sizes['medium'] );          // Remove Medium resolution (300 x 300 max height 300px)
	unset( $sizes['medium_large'] );    // Remove Medium Large (added in WP 4.4) resolution (768 x 0 infinite height)
	unset( $sizes['large'] );           // Remove Large resolution (1024 x 1024 max height 1024px)

	/* With WooCommerce */
	unset( $sizes[ 'shop_thumbnail' ]);  // Remove Shop thumbnail (180 x 180 hard cropped)
	unset( $sizes[ 'shop_catalog' ]);    // Remove Shop catalog (300 x 300 hard cropped)
	unset( $sizes[ 'shop_single' ]);     // Shop single (600 x 600 hard cropped)

	return $sizes;
}
add_filter( 'intermediate_image_sizes_advanced', 'remove_default_image_sizes' );
```

## Add CPT to category and tag archives

```php
/* ----------------------------------------------------------------------------
 * Add CPT to category and tag archives
 * ------------------------------------------------------------------------- */

function simple_add_custom_types( $query ) {
	if( is_category() || is_tag() && empty( $query->query_vars['suppress_filters'] ) ) {
		$post_types = get_post_types();
		$query->set( 'post_type', $post_type );
		return $query;
	}
}
add_filter( 'pre_get_posts', 'simple_add_custom_types');
```

## Add the support for your CPT in the widget activity of the admin dashboard

```php
/* ----------------------------------------------------------------------------
 * Add the support for your CPT in the widget activity of the admin dashboard
 * ------------------------------------------------------------------------- */

if ( is_admin() ) {
	add_filter( 'dashboard_recent_posts_query_args', 'add_page_to_dashboard_activity' );
	function add_page_to_dashboard_activity( $query ) {
		// Return all post types
		$post_types = get_post_types();
		// Return post types of your choice
		// $post_types = ['post', 'foo', 'bar'];
		if ( is_array( $query['post_type'] ) ) {
			$query['post_type'] = $post_types;
		} else {
			$temp = $post_types;
			$query['post_type'] = $temp;
		}
		return $query;
	}
}

```

## Remove all dashboard widgets

```php
/* ----------------------------------------------------------------------------
 * Remove all dashboard widgets
 * ------------------------------------------------------------------------- */

function remove_dashboard_widgets() {
	global $wp_meta_boxes;
	unset( $wp_meta_boxes['dashboard']['side']['core']['dashboard_quick_press'] );
	unset( $wp_meta_boxes['dashboard']['normal']['core']['dashboard_incoming_links'] );
	unset( $wp_meta_boxes['dashboard']['normal']['core']['dashboard_right_now'] );
	unset( $wp_meta_boxes['dashboard']['normal']['core']['dashboard_plugins'] );
	unset( $wp_meta_boxes['dashboard']['normal']['core']['dashboard_recent_drafts'] );
	unset( $wp_meta_boxes['dashboard']['normal']['core']['dashboard_recent_comments'] );
	unset( $wp_meta_boxes['dashboard']['side']['core']['dashboard_primary'] );
	unset( $wp_meta_boxes['dashboard']['side']['core']['dashboard_secondary'] );
	remove_meta_box( 'dashboard_activity', 'dashboard', 'normal' );
}
add_action( 'wp_dashboard_setup', 'remove_dashboard_widgets' );
```

## Remove welcome panel in dashboard

```php
/* ----------------------------------------------------------------------------
 * Remove welcome panel in dashboard
 * ------------------------------------------------------------------------- */

remove_action('welcome_panel', 'wp_welcome_panel');
```

## Let WordPress manage the document title

Add the following to functions.php and remove the `<title>` tag from your header.

```php
/* ----------------------------------------------------------------------------
 * Let WordPress manage the document title
 * ------------------------------------------------------------------------- */

add_theme_support( 'title-tag' );
```

## Switch default core markup to output valid HTML5

```php
/* ----------------------------------------------------------------------------
 * Switch default core markup to output valid HTML5
 * ------------------------------------------------------------------------- */

add_theme_support( 'html5', array(
	'search-form', 'comment-form', 'comment-list', 'gallery', 'caption',
) );
```

## Modify admin footer text

```php
/* ----------------------------------------------------------------------------
 * Modify admin footer text
 * ------------------------------------------------------------------------- */

function modify_admin_footer() {
	echo 'Created by <a href="mailto:you@domain.com">you</a>.';
}
add_filter( 'admin_footer_text', 'modify_admin_footer' );
```

## Modify excerpt length

```php
/* ----------------------------------------------------------------------------
 * Modify excerpt length
 * ------------------------------------------------------------------------- */

function custom_excerpt_length( $length ) {
	return 25;
}
add_filter( 'excerpt_length', 'custom_excerpt_length', 999 );
```

## Change more excerpt

```php
/* ----------------------------------------------------------------------------
 * Change more excerpt
 * ------------------------------------------------------------------------- */

function custom_more_excerpt( $more ) {
	return '...';
}
add_filter( 'excerpt_more', 'custom_more_excerpt' );
```

## Create WordPress settings page For custom options

For better maintainability of our code, let’s create a PHP class in separate file called `simple_settings_page.php` in theme folder.

```php
<?php
/* ----------------------------------------------------------------------------
 * Create WordPress settings page For custom options
 * ------------------------------------------------------------------------- */

// wp-content/themes/your-theme/simple_settings_page.php

class simple_settings_page {
	/**
	 * Array of custom settings/options
	**/
	private $options;

	/**
	 * Constructor
	 */
	public function __construct() {
		add_action( 'admin_menu', array( $this, 'add_settings_page' ) );
		add_action( 'admin_init', array( $this, 'page_init' ) );
	}

	/**
	 * Add settings page
	 * The page will appear in Admin menu
	 */
	public function add_settings_page() {
		add_menu_page(
			'Custom Settings', // Page title
			'Custom Settings Page', // Title
			'edit_pages', // Capability
			'custom-settings-page', // Url slug
			array( $this, 'create_admin_page' ) // Callback
		);
	}

	/**
	 * Options page callback
	 */
	public function create_admin_page() {
		// Set class property
		$this->options = get_option( 'custom_settings' );
		?>
		<div class="wrap">
			<h2>Custom settings page</h2>           
			<form method="post" action="options.php">
			<?php
				// This prints out all hidden setting fields
				settings_fields( 'custom_settings_group' );   
				do_settings_sections( 'custom-settings-page' );
				submit_button(); 
			?>
			</form>
		</div>
	<?php
	}

	/**
	 * Register and add settings
	 */
	public function page_init() {
		register_setting(
			'custom_settings_group', // Option group
			'custom_settings', // Option name
			array( $this, 'sanitize' ) // Sanitize
		);

		add_settings_section(
			'custom_settings_section', // ID
			'Custom Settings', // Title
			array( $this, 'custom_settings_section' ), // Callback
			'custom-settings-page' // Page
		);

		add_settings_field(
			'custom_setting_1', // ID
			'Custom Setting 1', // Title 
			array( $this, 'custom_setting1_html' ), // Callback
			'custom-settings-page', // Page         
			'custom_settings_section'
		);

		add_settings_field(
			'custom_setting_2', 
			'Custom Setting 2', 
			array( $this, 'custom_setting2_html' ), 
			'custom-settings-page',
			'custom_settings_section'
		);
	}

	/**
	 * Sanitize POST data from custom settings form
	 *
	 * @param array $input Contains custom settings which are passed when saving the form
	 */
	public function sanitize( $input ) {
		$sanitized_input= array();
		if( isset( $input['custom_setting_1'] ) )
			$sanitized_input['custom_setting_1'] = sanitize_text_field( $input['custom_setting_1'] );

		if( isset( $input['custom_setting_2'] ) )
			$sanitized_input['custom_setting_2'] = sanitize_text_field( $input['custom_setting_2'] );

		return $sanitized_input;
	}

	/** 
	 * Custom settings section text
	 */
	public function custom_settings_section() {
		print('Some text');
	}

	/** 
	 * HTML for custom setting 1 input
	 */
	public function custom_setting1_html() {
		printf(
			'<input type="text" id="custom_setting_1" name="custom_settings[custom_setting_1]" value="%s" />',
			isset( $this->options['custom_setting_1'] ) ? esc_attr( $this->options['custom_setting_1']) : ''
		);
	}

	/** 
	 * HTML for custom setting 2 input
	 */
	public function custom_setting2_html() {
		printf(
			'<input type="text" id="custom_setting_2" name="custom_settings[custom_setting_2]" value="%s" />',
			isset( $this->options['custom_setting_2'] ) ? esc_attr( $this->options['custom_setting_2']) : ''
		);
	}
}
```

Let’s instantiate this class in our functions.php file:

```php
if( is_admin() ) {
	require 'simple_settings_page.php';
	new simple_settings_page();
}
```

When you will want to get those options somewhere else, just use get_option function, like this:

```php
$custom_settings = get_option( 'custom_settings' );
$custom_setting_1 = $custom_settings['custom_setting_1'];
$custom_setting_2 = $custom_settings['custom_setting_2'];
```

## Add Bootstrap 3 navigation style to WordPress menu manager

[WP Bootstrap Navwalker](https://github.com/wp-bootstrap/wp-bootstrap-navwalker)

A custom WordPress nav walker class to implement the Bootstrap 3 navigation style in a custom theme using the WordPress built in menu manager.

### License
[MIT License](LICENSE) © Cool WP Functions

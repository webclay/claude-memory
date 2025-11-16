# WordPress Plugin Development - Patterns & Standards

**Purpose:** Comprehensive guide for developing WordPress plugins
**Last Updated:** 2025-01-12
**WordPress Version:** 6.4+
**PHP Version:** 8.0+

---

## Overview

WordPress plugin development requires understanding WordPress core APIs, hooks, database patterns, and security best practices. This guide covers modern WordPress plugin development with a focus on maintainability and WordPress coding standards.

**Key Concepts:**
- Actions and Filters (WordPress Hook System)
- Custom Post Types and Taxonomies
- WordPress REST API
- Database operations with wpdb
- Security (nonces, sanitization, escaping)
- Admin interfaces and settings
- Gutenberg blocks (optional)

---

## Plugin Structure

### Basic Plugin File Structure

```
my-plugin/
├── my-plugin.php              # Main plugin file (required)
├── uninstall.php              # Cleanup on uninstall (optional)
├── README.txt                 # WordPress.org readme (if publishing)
├── includes/
│   ├── class-plugin-name.php  # Main plugin class
│   ├── class-activator.php    # Activation hooks
│   ├── class-deactivator.php  # Deactivation hooks
│   └── class-database.php     # Database operations
├── admin/
│   ├── class-admin.php        # Admin-specific functionality
│   ├── partials/              # Admin view templates
│   │   └── admin-display.php
│   ├── css/
│   │   └── admin-styles.css
│   └── js/
│       └── admin-scripts.js
├── public/
│   ├── class-public.php       # Public-facing functionality
│   ├── css/
│   │   └── public-styles.css
│   └── js/
│       └── public-scripts.js
├── api/
│   └── class-rest-api.php     # REST API endpoints
└── blocks/                    # Gutenberg blocks (if applicable)
    └── booking-form/
        ├── block.json
        ├── index.js
        └── style.css
```

---

## Main Plugin File

### Basic Plugin Header

Every WordPress plugin starts with a main file with proper headers:

```php
<?php
/**
 * Plugin Name:       My Awesome Plugin
 * Plugin URI:        https://example.com/my-awesome-plugin
 * Description:       A brief description of what your plugin does
 * Version:           1.0.0
 * Requires at least: 6.0
 * Requires PHP:      8.0
 * Author:            Your Name
 * Author URI:        https://example.com
 * License:           GPL v2 or later
 * License URI:       https://www.gnu.org/licenses/gpl-2.0.html
 * Text Domain:       my-awesome-plugin
 * Domain Path:       /languages
 */

// If this file is called directly, abort.
if ( ! defined( 'WPINC' ) ) {
    die;
}

// Define plugin constants
define( 'MY_PLUGIN_VERSION', '1.0.0' );
define( 'MY_PLUGIN_PATH', plugin_dir_path( __FILE__ ) );
define( 'MY_PLUGIN_URL', plugin_dir_url( __FILE__ ) );
define( 'MY_PLUGIN_BASENAME', plugin_basename( __FILE__ ) );

/**
 * Activation hook
 */
function activate_my_plugin() {
    require_once MY_PLUGIN_PATH . 'includes/class-activator.php';
    My_Plugin_Activator::activate();
}
register_activation_hook( __FILE__, 'activate_my_plugin' );

/**
 * Deactivation hook
 */
function deactivate_my_plugin() {
    require_once MY_PLUGIN_PATH . 'includes/class-deactivator.php';
    My_Plugin_Deactivator::deactivate();
}
register_deactivation_hook( __FILE__, 'deactivate_my_plugin' );

/**
 * Initialize plugin
 */
function run_my_plugin() {
    require_once MY_PLUGIN_PATH . 'includes/class-plugin-name.php';
    $plugin = new My_Plugin();
    $plugin->run();
}
run_my_plugin();
```

### Plugin Activation

Create `includes/class-activator.php`:

```php
<?php
/**
 * Fired during plugin activation
 */
class My_Plugin_Activator {

    /**
     * Activation tasks
     */
    public static function activate() {
        // Create custom database tables
        self::create_tables();

        // Create default options
        self::set_default_options();

        // Flush rewrite rules (for custom post types)
        flush_rewrite_rules();

        // Set plugin version
        update_option( 'my_plugin_version', MY_PLUGIN_VERSION );
    }

    /**
     * Create custom database tables
     */
    private static function create_tables() {
        global $wpdb;

        $charset_collate = $wpdb->get_charset_collate();
        $table_name = $wpdb->prefix . 'my_plugin_data';

        $sql = "CREATE TABLE $table_name (
            id bigint(20) NOT NULL AUTO_INCREMENT,
            user_id bigint(20) NOT NULL,
            data_field varchar(255) NOT NULL,
            created_at datetime DEFAULT CURRENT_TIMESTAMP,
            updated_at datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
            PRIMARY KEY  (id),
            KEY user_id (user_id)
        ) $charset_collate;";

        require_once( ABSPATH . 'wp-admin/includes/upgrade.php' );
        dbDelta( $sql );
    }

    /**
     * Set default plugin options
     */
    private static function set_default_options() {
        $defaults = array(
            'enable_feature' => true,
            'api_key' => '',
            'items_per_page' => 10,
        );

        add_option( 'my_plugin_settings', $defaults );
    }
}
```

### Plugin Deactivation

Create `includes/class-deactivator.php`:

```php
<?php
/**
 * Fired during plugin deactivation
 */
class My_Plugin_Deactivator {

    /**
     * Deactivation tasks
     */
    public static function deactivate() {
        // Flush rewrite rules
        flush_rewrite_rules();

        // Clear scheduled cron jobs
        wp_clear_scheduled_hook( 'my_plugin_daily_task' );

        // Note: Don't delete user data on deactivation
        // Only delete data in uninstall.php
    }
}
```

---

## WordPress Hooks System

### Actions vs Filters

**Actions:** Do something at a specific point
**Filters:** Modify data before it's used

```php
// ACTION: Do something
add_action( 'init', 'my_custom_function' );
function my_custom_function() {
    // Register custom post type, enqueue scripts, etc.
}

// FILTER: Modify content
add_filter( 'the_content', 'add_custom_content' );
function add_custom_content( $content ) {
    $custom = '<p>Custom content</p>';
    return $content . $custom;
}
```

### Common WordPress Hooks

```php
// Plugin initialization
add_action( 'plugins_loaded', 'my_plugin_init' );

// Admin initialization
add_action( 'admin_init', 'my_admin_init' );

// Enqueue scripts (public)
add_action( 'wp_enqueue_scripts', 'my_enqueue_public_scripts' );

// Enqueue scripts (admin)
add_action( 'admin_enqueue_scripts', 'my_enqueue_admin_scripts' );

// Save post hook
add_action( 'save_post', 'my_save_post_meta', 10, 2 );

// User registration
add_action( 'user_register', 'my_new_user_action' );

// Modify post content
add_filter( 'the_content', 'my_modify_content' );

// Modify post title
add_filter( 'the_title', 'my_modify_title', 10, 2 );
```

### Custom Hooks (For Extensibility)

```php
// In your plugin, create custom hooks for other developers

// Custom action
do_action( 'my_plugin_before_save', $data );
// Process data
do_action( 'my_plugin_after_save', $saved_data );

// Custom filter
$modified_data = apply_filters( 'my_plugin_process_data', $data );
```

---

## Database Operations with wpdb

### Using $wpdb Safely

```php
global $wpdb;

// ✅ CORRECT: Use prepare() to prevent SQL injection
$user_id = 123;
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}my_plugin_data WHERE user_id = %d",
        $user_id
    )
);

// ❌ WRONG: Never use direct user input
// $wpdb->query( "SELECT * FROM table WHERE id = " . $_GET['id'] );
```

### Common wpdb Methods

**SELECT Queries:**

```php
global $wpdb;

// Get multiple rows
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}my_plugin_data WHERE user_id = %d",
        $user_id
    ),
    ARRAY_A // or OBJECT (default)
);

// Get single row
$row = $wpdb->get_row(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}my_plugin_data WHERE id = %d",
        $id
    )
);

// Get single value
$count = $wpdb->get_var(
    "SELECT COUNT(*) FROM {$wpdb->prefix}my_plugin_data"
);

// Get single column
$ids = $wpdb->get_col(
    "SELECT id FROM {$wpdb->prefix}my_plugin_data"
);
```

**INSERT:**

```php
global $wpdb;

$wpdb->insert(
    $wpdb->prefix . 'my_plugin_data',
    array(
        'user_id' => $user_id,
        'data_field' => $data,
        'created_at' => current_time( 'mysql' ),
    ),
    array( '%d', '%s', '%s' ) // Data format
);

$inserted_id = $wpdb->insert_id;
```

**UPDATE:**

```php
global $wpdb;

$wpdb->update(
    $wpdb->prefix . 'my_plugin_data',
    array( 'data_field' => $new_data ), // Data to update
    array( 'id' => $id ), // WHERE clause
    array( '%s' ), // Data format
    array( '%d' ) // WHERE format
);

$rows_affected = $wpdb->rows_affected;
```

**DELETE:**

```php
global $wpdb;

$wpdb->delete(
    $wpdb->prefix . 'my_plugin_data',
    array( 'id' => $id ),
    array( '%d' )
);
```

### Using WordPress Post Meta (Simpler Alternative)

For simple key-value storage tied to posts/users:

```php
// Add meta
add_post_meta( $post_id, 'my_custom_field', $value );

// Update meta
update_post_meta( $post_id, 'my_custom_field', $new_value );

// Get meta
$value = get_post_meta( $post_id, 'my_custom_field', true );

// Delete meta
delete_post_meta( $post_id, 'my_custom_field' );

// User meta (same pattern)
update_user_meta( $user_id, 'my_user_field', $value );
$value = get_user_meta( $user_id, 'my_user_field', true );
```

---

## Custom Post Types

### Registering Custom Post Type

```php
add_action( 'init', 'register_booking_post_type' );

function register_booking_post_type() {
    $labels = array(
        'name'               => 'Bookings',
        'singular_name'      => 'Booking',
        'add_new'            => 'Add New',
        'add_new_item'       => 'Add New Booking',
        'edit_item'          => 'Edit Booking',
        'new_item'           => 'New Booking',
        'view_item'          => 'View Booking',
        'search_items'       => 'Search Bookings',
        'not_found'          => 'No bookings found',
        'not_found_in_trash' => 'No bookings found in Trash',
    );

    $args = array(
        'labels'              => $labels,
        'public'              => true,
        'publicly_queryable'  => true,
        'show_ui'             => true,
        'show_in_menu'        => true,
        'show_in_rest'        => true, // Enable Gutenberg & REST API
        'query_var'           => true,
        'rewrite'             => array( 'slug' => 'bookings' ),
        'capability_type'     => 'post',
        'has_archive'         => true,
        'hierarchical'        => false,
        'menu_position'       => 20,
        'menu_icon'           => 'dashicons-calendar-alt',
        'supports'            => array( 'title', 'editor', 'thumbnail', 'custom-fields' ),
    );

    register_post_type( 'booking', $args );
}
```

### Custom Taxonomy

```php
add_action( 'init', 'register_booking_category_taxonomy' );

function register_booking_category_taxonomy() {
    $labels = array(
        'name'              => 'Booking Categories',
        'singular_name'     => 'Booking Category',
        'search_items'      => 'Search Categories',
        'all_items'         => 'All Categories',
        'parent_item'       => 'Parent Category',
        'parent_item_colon' => 'Parent Category:',
        'edit_item'         => 'Edit Category',
        'update_item'       => 'Update Category',
        'add_new_item'      => 'Add New Category',
        'new_item_name'     => 'New Category Name',
        'menu_name'         => 'Categories',
    );

    $args = array(
        'hierarchical'      => true, // Like categories
        'labels'            => $labels,
        'show_ui'           => true,
        'show_admin_column' => true,
        'query_var'         => true,
        'show_in_rest'      => true,
        'rewrite'           => array( 'slug' => 'booking-category' ),
    );

    register_taxonomy( 'booking_category', array( 'booking' ), $args );
}
```

---

## WordPress REST API

### Registering Custom Endpoints

```php
add_action( 'rest_api_init', 'register_my_plugin_endpoints' );

function register_my_plugin_endpoints() {
    // GET endpoint
    register_rest_route( 'my-plugin/v1', '/bookings', array(
        'methods'             => 'GET',
        'callback'            => 'get_bookings',
        'permission_callback' => 'my_plugin_permissions_check',
    ) );

    // POST endpoint
    register_rest_route( 'my-plugin/v1', '/bookings', array(
        'methods'             => 'POST',
        'callback'            => 'create_booking',
        'permission_callback' => 'my_plugin_permissions_check',
        'args'                => array(
            'title' => array(
                'required'          => true,
                'validate_callback' => function( $param ) {
                    return is_string( $param );
                },
                'sanitize_callback' => 'sanitize_text_field',
            ),
            'date' => array(
                'required'          => true,
                'validate_callback' => function( $param ) {
                    return strtotime( $param ) !== false;
                },
            ),
        ),
    ) );

    // GET single item
    register_rest_route( 'my-plugin/v1', '/bookings/(?P<id>\d+)', array(
        'methods'             => 'GET',
        'callback'            => 'get_single_booking',
        'permission_callback' => 'my_plugin_permissions_check',
        'args'                => array(
            'id' => array(
                'validate_callback' => function( $param ) {
                    return is_numeric( $param );
                },
            ),
        ),
    ) );

    // DELETE endpoint
    register_rest_route( 'my-plugin/v1', '/bookings/(?P<id>\d+)', array(
        'methods'             => 'DELETE',
        'callback'            => 'delete_booking',
        'permission_callback' => 'my_plugin_delete_permissions_check',
    ) );
}

/**
 * Permission callback
 */
function my_plugin_permissions_check() {
    return current_user_can( 'read' );
}

function my_plugin_delete_permissions_check() {
    return current_user_can( 'delete_posts' );
}

/**
 * GET callback
 */
function get_bookings( $request ) {
    global $wpdb;

    $results = $wpdb->get_results(
        "SELECT * FROM {$wpdb->prefix}my_plugin_data ORDER BY created_at DESC",
        ARRAY_A
    );

    return new WP_REST_Response( $results, 200 );
}

/**
 * POST callback
 */
function create_booking( $request ) {
    global $wpdb;

    $title = $request->get_param( 'title' );
    $date = $request->get_param( 'date' );

    $inserted = $wpdb->insert(
        $wpdb->prefix . 'my_plugin_data',
        array(
            'user_id'    => get_current_user_id(),
            'data_field' => $title,
            'created_at' => $date,
        ),
        array( '%d', '%s', '%s' )
    );

    if ( $inserted ) {
        return new WP_REST_Response(
            array(
                'id'      => $wpdb->insert_id,
                'message' => 'Booking created successfully',
            ),
            201
        );
    } else {
        return new WP_Error(
            'booking_creation_failed',
            'Failed to create booking',
            array( 'status' => 500 )
        );
    }
}

/**
 * GET single item callback
 */
function get_single_booking( $request ) {
    global $wpdb;

    $id = $request->get_param( 'id' );

    $result = $wpdb->get_row(
        $wpdb->prepare(
            "SELECT * FROM {$wpdb->prefix}my_plugin_data WHERE id = %d",
            $id
        ),
        ARRAY_A
    );

    if ( $result ) {
        return new WP_REST_Response( $result, 200 );
    } else {
        return new WP_Error(
            'booking_not_found',
            'Booking not found',
            array( 'status' => 404 )
        );
    }
}

/**
 * DELETE callback
 */
function delete_booking( $request ) {
    global $wpdb;

    $id = $request->get_param( 'id' );

    $deleted = $wpdb->delete(
        $wpdb->prefix . 'my_plugin_data',
        array( 'id' => $id ),
        array( '%d' )
    );

    if ( $deleted ) {
        return new WP_REST_Response(
            array( 'message' => 'Booking deleted successfully' ),
            200
        );
    } else {
        return new WP_Error(
            'deletion_failed',
            'Failed to delete booking',
            array( 'status' => 500 )
        );
    }
}
```

### Calling REST API from JavaScript

```javascript
// Using wp.apiFetch (recommended in WordPress admin)
wp.apiFetch({
    path: '/my-plugin/v1/bookings',
    method: 'POST',
    data: {
        title: 'New Booking',
        date: '2025-01-15'
    }
}).then(response => {
    console.log('Booking created:', response);
}).catch(error => {
    console.error('Error:', error);
});

// Using fetch API (for frontend)
fetch('/wp-json/my-plugin/v1/bookings', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-WP-Nonce': wpApiSettings.nonce // Pass nonce for auth
    },
    body: JSON.stringify({
        title: 'New Booking',
        date: '2025-01-15'
    })
})
.then(response => response.json())
.then(data => console.log(data));
```

---

## Security in WordPress

### 1. Nonces (Prevent CSRF)

```php
// Create nonce
$nonce = wp_create_nonce( 'my_action_nonce' );

// In form
<input type="hidden" name="my_nonce" value="<?php echo $nonce; ?>" />

// Verify nonce
if ( ! wp_verify_nonce( $_POST['my_nonce'], 'my_action_nonce' ) ) {
    die( 'Security check failed' );
}

// For AJAX
wp_localize_script( 'my-script', 'myAjax', array(
    'ajaxurl' => admin_url( 'admin-ajax.php' ),
    'nonce'   => wp_create_nonce( 'my_ajax_nonce' ),
) );

// Verify in AJAX handler
check_ajax_referer( 'my_ajax_nonce', 'nonce' );
```

### 2. Data Sanitization (Input)

```php
// Text fields
$clean_text = sanitize_text_field( $_POST['field'] );

// Textarea
$clean_textarea = sanitize_textarea_field( $_POST['message'] );

// Email
$clean_email = sanitize_email( $_POST['email'] );

// URL
$clean_url = esc_url_raw( $_POST['url'] );

// Integer
$clean_int = absint( $_POST['id'] );

// Array
$clean_array = array_map( 'sanitize_text_field', $_POST['items'] );
```

### 3. Data Escaping (Output)

```php
// General text
echo esc_html( $text );

// Attributes
echo '<a href="' . esc_url( $url ) . '">' . esc_html( $title ) . '</a>';

// JavaScript
echo '<script>var data = ' . wp_json_encode( $data ) . ';</script>';

// Textarea
echo '<textarea>' . esc_textarea( $content ) . '</textarea>';

// Allow some HTML
echo wp_kses_post( $content ); // Allows post content HTML
```

### 4. Capability Checks

```php
// Check if user can perform action
if ( ! current_user_can( 'manage_options' ) ) {
    wp_die( 'Unauthorized' );
}

// Common capabilities:
// - read (all users)
// - edit_posts (contributors and above)
// - publish_posts (authors and above)
// - edit_pages (editors and above)
// - manage_options (administrators)
```

---

## Admin Pages & Settings

### Add Admin Menu Page

```php
add_action( 'admin_menu', 'my_plugin_add_admin_menu' );

function my_plugin_add_admin_menu() {
    add_menu_page(
        'My Plugin Settings',           // Page title
        'My Plugin',                    // Menu title
        'manage_options',               // Capability
        'my-plugin',                    // Menu slug
        'my_plugin_settings_page',      // Callback function
        'dashicons-admin-generic',      // Icon
        80                              // Position
    );

    // Add submenu page
    add_submenu_page(
        'my-plugin',                    // Parent slug
        'Advanced Settings',            // Page title
        'Advanced',                     // Menu title
        'manage_options',               // Capability
        'my-plugin-advanced',           // Menu slug
        'my_plugin_advanced_page'       // Callback
    );
}

function my_plugin_settings_page() {
    ?>
    <div class="wrap">
        <h1><?php echo esc_html( get_admin_page_title() ); ?></h1>
        <form action="options.php" method="post">
            <?php
            settings_fields( 'my_plugin_settings' );
            do_settings_sections( 'my-plugin' );
            submit_button();
            ?>
        </form>
    </div>
    <?php
}
```

### Register Settings

```php
add_action( 'admin_init', 'my_plugin_register_settings' );

function my_plugin_register_settings() {
    register_setting(
        'my_plugin_settings',     // Option group
        'my_plugin_options',      // Option name
        'my_plugin_sanitize_options' // Sanitize callback
    );

    add_settings_section(
        'my_plugin_main_section',
        'Main Settings',
        'my_plugin_section_callback',
        'my-plugin'
    );

    add_settings_field(
        'api_key',
        'API Key',
        'my_plugin_api_key_field',
        'my-plugin',
        'my_plugin_main_section'
    );

    add_settings_field(
        'enable_feature',
        'Enable Feature',
        'my_plugin_enable_feature_field',
        'my-plugin',
        'my_plugin_main_section'
    );
}

function my_plugin_section_callback() {
    echo '<p>Configure your plugin settings below.</p>';
}

function my_plugin_api_key_field() {
    $options = get_option( 'my_plugin_options' );
    $value = isset( $options['api_key'] ) ? $options['api_key'] : '';
    ?>
    <input type="text"
           name="my_plugin_options[api_key]"
           value="<?php echo esc_attr( $value ); ?>"
           class="regular-text" />
    <?php
}

function my_plugin_enable_feature_field() {
    $options = get_option( 'my_plugin_options' );
    $checked = isset( $options['enable_feature'] ) && $options['enable_feature'];
    ?>
    <input type="checkbox"
           name="my_plugin_options[enable_feature]"
           value="1"
           <?php checked( $checked, true ); ?> />
    <?php
}

function my_plugin_sanitize_options( $input ) {
    $sanitized = array();

    if ( isset( $input['api_key'] ) ) {
        $sanitized['api_key'] = sanitize_text_field( $input['api_key'] );
    }

    $sanitized['enable_feature'] = isset( $input['enable_feature'] ) ? true : false;

    return $sanitized;
}
```

---

## Enqueuing Scripts & Styles

### Proper Enqueueing

```php
// Frontend scripts
add_action( 'wp_enqueue_scripts', 'my_plugin_enqueue_public_assets' );

function my_plugin_enqueue_public_assets() {
    // CSS
    wp_enqueue_style(
        'my-plugin-public-styles',
        MY_PLUGIN_URL . 'public/css/public-styles.css',
        array(),
        MY_PLUGIN_VERSION,
        'all'
    );

    // JavaScript
    wp_enqueue_script(
        'my-plugin-public-scripts',
        MY_PLUGIN_URL . 'public/js/public-scripts.js',
        array( 'jquery' ), // Dependencies
        MY_PLUGIN_VERSION,
        true // Load in footer
    );

    // Pass data to JavaScript
    wp_localize_script(
        'my-plugin-public-scripts',
        'myPluginData',
        array(
            'ajaxUrl' => admin_url( 'admin-ajax.php' ),
            'nonce'   => wp_create_nonce( 'my_plugin_nonce' ),
            'userId'  => get_current_user_id(),
        )
    );
}

// Admin scripts
add_action( 'admin_enqueue_scripts', 'my_plugin_enqueue_admin_assets' );

function my_plugin_enqueue_admin_assets( $hook ) {
    // Only load on our plugin pages
    if ( strpos( $hook, 'my-plugin' ) === false ) {
        return;
    }

    wp_enqueue_style(
        'my-plugin-admin-styles',
        MY_PLUGIN_URL . 'admin/css/admin-styles.css',
        array(),
        MY_PLUGIN_VERSION
    );

    wp_enqueue_script(
        'my-plugin-admin-scripts',
        MY_PLUGIN_URL . 'admin/js/admin-scripts.js',
        array( 'jquery' ),
        MY_PLUGIN_VERSION,
        true
    );
}
```

---

## AJAX Handling

### PHP - Register AJAX Handlers

```php
// For logged-in users
add_action( 'wp_ajax_my_plugin_action', 'my_plugin_ajax_handler' );

// For non-logged-in users
add_action( 'wp_ajax_nopriv_my_plugin_action', 'my_plugin_ajax_handler' );

function my_plugin_ajax_handler() {
    // Verify nonce
    check_ajax_referer( 'my_plugin_nonce', 'nonce' );

    // Get data
    $data = isset( $_POST['data'] ) ? sanitize_text_field( $_POST['data'] ) : '';

    // Process data
    $result = process_my_data( $data );

    // Return response
    if ( $result ) {
        wp_send_json_success(
            array( 'message' => 'Success!' )
        );
    } else {
        wp_send_json_error(
            array( 'message' => 'Something went wrong' )
        );
    }
}
```

### JavaScript - Make AJAX Request

```javascript
jQuery(document).ready(function($) {
    $('#my-button').on('click', function() {
        $.ajax({
            url: myPluginData.ajaxUrl,
            type: 'POST',
            data: {
                action: 'my_plugin_action',
                nonce: myPluginData.nonce,
                data: 'some data'
            },
            success: function(response) {
                if (response.success) {
                    console.log(response.data.message);
                } else {
                    console.error(response.data.message);
                }
            },
            error: function(xhr, status, error) {
                console.error('AJAX error:', error);
            }
        });
    });
});
```

---

## Shortcodes

### Register Shortcode

```php
add_shortcode( 'my_booking_form', 'my_booking_form_shortcode' );

function my_booking_form_shortcode( $atts ) {
    // Parse attributes
    $atts = shortcode_atts(
        array(
            'type' => 'default',
            'limit' => 10,
        ),
        $atts,
        'my_booking_form'
    );

    // Start output buffering
    ob_start();

    ?>
    <div class="my-booking-form">
        <h3>Book Now</h3>
        <form id="booking-form">
            <input type="text" name="name" placeholder="Your Name" required />
            <input type="email" name="email" placeholder="Your Email" required />
            <input type="date" name="date" required />
            <button type="submit">Submit Booking</button>
        </form>
    </div>
    <?php

    return ob_get_clean();
}

// Usage in posts/pages: [my_booking_form type="premium" limit="5"]
```

---

## WordPress Coding Standards

### Naming Conventions

```php
// Functions: lowercase with underscores
function my_plugin_function_name() {}

// Classes: First letter capitalized, underscores
class My_Plugin_Class_Name {}

// Constants: ALL CAPS with underscores
define( 'MY_PLUGIN_VERSION', '1.0.0' );

// Variables: lowercase with underscores
$my_variable = 'value';
```

### File Structure

```php
<?php
/**
 * File description
 *
 * @package My_Plugin
 */

// Prevent direct access
if ( ! defined( 'ABSPATH' ) ) {
    exit;
}

// Code here
```

### Code Formatting

```php
// Spaces after commas
function my_function( $arg1, $arg2, $arg3 ) {
    // Indentation: 4 spaces or 1 tab
    if ( $condition ) {
        do_something();
    } else {
        do_something_else();
    }
}

// Yoda conditions (for comparison)
if ( 'value' === $variable ) { // Prevents accidental assignment
    // Do something
}
```

---

## Uninstall Cleanup

Create `uninstall.php` in plugin root:

```php
<?php
/**
 * Fired when the plugin is uninstalled
 */

// If uninstall not called from WordPress, exit
if ( ! defined( 'WP_UNINSTALL_PLUGIN' ) ) {
    exit;
}

global $wpdb;

// Delete custom tables
$table_name = $wpdb->prefix . 'my_plugin_data';
$wpdb->query( "DROP TABLE IF EXISTS $table_name" );

// Delete options
delete_option( 'my_plugin_settings' );
delete_option( 'my_plugin_version' );

// Delete post meta
$wpdb->query( "DELETE FROM $wpdb->postmeta WHERE meta_key LIKE 'my_plugin_%'" );

// Delete user meta
$wpdb->query( "DELETE FROM $wpdb->usermeta WHERE meta_key LIKE 'my_plugin_%'" );

// Clear scheduled hooks
wp_clear_scheduled_hook( 'my_plugin_daily_task' );
```

---

## Testing WordPress Plugins

### Manual Testing Checklist

- [ ] Test plugin activation/deactivation
- [ ] Test on fresh WordPress installation
- [ ] Test with different themes
- [ ] Test with common plugins (WooCommerce, Yoast, etc.)
- [ ] Test with PHP 8.0, 8.1, 8.2
- [ ] Test on different WordPress versions (6.0+)
- [ ] Test all AJAX functionality
- [ ] Test all REST API endpoints
- [ ] Verify all data is sanitized and escaped
- [ ] Check for PHP errors/warnings
- [ ] Test uninstall cleanup

### WordPress Coding Standards Check

```bash
# Install PHP_CodeSniffer
composer global require "squizlabs/php_codesniffer=*"

# Install WordPress Coding Standards
composer global require wp-coding-standards/wpcs

# Run check
phpcs --standard=WordPress /path/to/plugin
```

---

## Common Patterns

### Admin Notice

```php
add_action( 'admin_notices', 'my_plugin_admin_notice' );

function my_plugin_admin_notice() {
    ?>
    <div class="notice notice-success is-dismissible">
        <p><?php esc_html_e( 'Plugin activated successfully!', 'my-plugin' ); ?></p>
    </div>
    <?php
}
```

### Cron Jobs

```php
// Schedule event on activation
register_activation_hook( __FILE__, 'my_plugin_schedule_event' );

function my_plugin_schedule_event() {
    if ( ! wp_next_scheduled( 'my_plugin_daily_task' ) ) {
        wp_schedule_event( time(), 'daily', 'my_plugin_daily_task' );
    }
}

// Hook to scheduled event
add_action( 'my_plugin_daily_task', 'my_plugin_do_daily_task' );

function my_plugin_do_daily_task() {
    // Your daily task code
}

// Clear on deactivation
register_deactivation_hook( __FILE__, 'my_plugin_clear_schedule' );

function my_plugin_clear_schedule() {
    wp_clear_scheduled_hook( 'my_plugin_daily_task' );
}
```

---

## Resources

- **WordPress Plugin Handbook:** https://developer.wordpress.org/plugins/
- **WordPress Coding Standards:** https://developer.wordpress.org/coding-standards/
- **WordPress REST API:** https://developer.wordpress.org/rest-api/
- **WordPress Database Class:** https://developer.wordpress.org/reference/classes/wpdb/
- **Plugin Boilerplate Generator:** https://wppb.me/

---

*This guide covers core WordPress plugin development patterns. For Gutenberg block development, advanced hooks, or specific integrations, consult the WordPress Developer Documentation.*

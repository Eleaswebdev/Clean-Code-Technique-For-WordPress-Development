# Clean-Code-Technique-For-WordPress-Development
Clean Code Technique For WordPress Development
Clean Code Technique:


1. Meaningful Naming Conventions
Principle: Use clear, descriptive names for variables, functions, and classes.
Example:
function get_published_posts_by_category($category_id) {
    $args = [
        'post_type'   => 'post',
        'category__in' => [$category_id],
        'post_status' => 'publish',
        'posts_per_page' => -1
    ];
    return get_posts($args);
}

Bad Example:
function getData($id) {
    return get_posts(['category' => $id]);
}

Why: 
getData is vague, whereas get_published_posts_by_category clearly describes what the function does.

2. Single Responsibility Principle (SRP)
Principle: A function or class should have only one reason to change.

Example:
class PostManager {
    public function get_published_posts($category_id) {
        return get_posts([
            'post_type'   => 'post',
            'category__in' => [$category_id],
            'post_status' => 'publish',
            'posts_per_page' => -1
        ]);
    }
}

Bad Example:

class PostManager {
    public function get_published_posts($category_id) {
        $posts = get_posts(['category' => $category_id]);
        echo '<ul>';
        foreach ($posts as $post) {
            echo '<li>' . $post->post_title . '</li>';
        }
        echo '</ul>';
    }
}

Why:
The second class mixes fetching and rendering posts, violating SRP. The first class only handles data fetching.

3. Open-Closed Principle (OCP)
Principle: A class should be open for extension but closed for modification.

Example:
abstract class Discount {
    abstract public function calculate($price);
}

class PercentageDiscount extends Discount {
    private $percentage;
    public function __construct($percentage) {
        $this->percentage = $percentage;
    }
    public function calculate($price) {
        return $price - ($price * ($this->percentage / 100));
    }
}

Bad Example:
class Discount {
    public function calculate($price, $type, $value) {
        if ($type === 'percentage') {
            return $price - ($price * ($value / 100));
        } elseif ($type === 'fixed') {
            return $price - $value;
        }
        return $price;
    }
}

Why:
The second example requires modifying the calculate method for every new discount type, while the first allows extending without modifying existing code.

4. DRY (Don’t Repeat Yourself)
Principle: Avoid code duplication by reusing functions or classes.


Example:

function format_price($price) {
    return number_format($price, 2, '.', ',') . ' USD';
}

function display_product_price($product_id) {
    $price = get_post_meta($product_id, '_price', true);
    echo format_price($price);
}

Bad Example:

function display_product_price($product_id) {
    $price = get_post_meta($product_id, '_price', true);
    echo number_format($price, 2, '.', ',') . ' USD';
}
function display_cart_total($total) {
    echo number_format($total, 2, '.', ',') . ' USD';
}

Why:
The second example repeats the number_format logic. The first reuses a helper function.


5. KISS (Keep It Simple, Stupid)

Principle: Write simple and straightforward code.

Example:

function is_user_admin($user_id) {
    return user_can($user_id, 'administrator');
}

Bad Example:

function is_user_admin($user_id) {
    $user = get_userdata($user_id);
    if ($user) {
        $roles = $user->roles;
        if (in_array('administrator', $roles)) {
            return true;
        }
    }
    return false;
}

Why:
The first example is more concise and achieves the same goal.


6. YAGNI (You Ain’t Gonna Need It)

Principle: Don’t add unnecessary functionality before it's needed.

Example:

function get_user_email($user_id) {
    return get_userdata($user_id)->user_email;
}

Bad Example:

function get_user_email($user_id, $return_json = false) {
    $email = get_userdata($user_id)->user_email;
    if ($return_json) {
        return json_encode(['email' => $email]);
    }
    return $email;
}

Why:
The second function adds unnecessary JSON formatting, which may not be required.


7. Avoid Hardcoding (Use Constants or Options)

Principle: Define values dynamically instead of hardcoding.




Example:

define('MY_THEME_VERSION', '1.0.0');
function get_theme_version() {
    return MY_THEME_VERSION;
}

Bad Example:

function get_theme_version() {
    return '1.0.0';
}

Why:
Using constants makes it easier to update values.

8. Proper Error Handling
Principle: Handle errors properly instead of suppressing them.

Example:

function get_custom_meta($post_id, $key) {
    $value = get_post_meta($post_id, $key, true);
    if (empty($value)) {
        return new WP_Error('not_found', 'Meta key not found');
    }
    return $value;
}

Bad Example:

function get_custom_meta($post_id, $key) {
    return get_post_meta($post_id, $key, true) ?: 'N/A';
}

Why:
Using WP_Error provides meaningful error handling.





9. Dependency Injection (DI)
Principle: Pass dependencies instead of instantiating them inside a class.

Example:

class Logger {
    public function log($message) {
        error_log($message);
    }
}

class Order {
    private $logger;
    public function __construct(Logger $logger) {
        $this->logger = $logger;
    }
    public function place_order() {
        $this->logger->log("Order placed.");
    }
}

Bad Example:

class Order {
    private $logger;
    public function __construct() {
        $this->logger = new Logger();
    }
    public function place_order() {
        $this->logger->log("Order placed.");
    }
}

Why:
The first allows for easy testing and modification of dependencies.









10. Avoid Deep Nesting
Principle: Avoid too many nested conditions, which reduce readability.

Example:

if (!$user || !$user->is_admin || !$user->has_permission) {
    return;
}

echo "Access granted.";

Bad Example:

if ($user) {
    if ($user->is_admin) {
        if ($user->has_permission) {
            echo "Access granted.";
        }
    }
}

Why:
The second example is easier to read.

11. Use Hooks Instead of Direct Code Execution

Principle: Use WordPress hooks (actions & filters) instead of modifying core behavior directly.

Example:

function restrict_admin_access() {
    if (!current_user_can('administrator')) {
        wp_die("You are not allowed here.");
    }
}
add_action('admin_init', 'restrict_admin_access');





Bad Example:

function restrict_admin_access() {
    if (!current_user_can('administrator')) {
        wp_die("You are not allowed here.");
    }
}
restrict_admin_access();

Why:
Using hooks makes the function modular and reusable.

12. Favor isset() & empty() Over Direct Checks
Principle: Avoid undefined index warnings.

Example:

if (!empty($_GET['page']) && $_GET['page'] === 'settings') {
    echo "Settings Page";
}

Bad Example:

if ($_GET['page'] === 'settings') {
    echo "Settings Page";
}

Why:
Prevents "Undefined index" notices.

13. Use Prepared Statements to Prevent SQL Injection
Principle: Always sanitize database queries.

Example:

global $wpdb;
$email = sanitize_email($_GET['email']);
$results = $wpdb->get_results($wpdb->prepare("SELECT * FROM wp_users WHERE user_email = %s", $email));


Bad Example:

global $wpdb;
$results = $wpdb->get_results("SELECT * FROM wp_users WHERE user_email = '{$_GET['email']}'");

Why:
 Prevents SQL injection.

14. Use wp_enqueue_script() and wp_enqueue_style()

15. Use wp_safe_redirect() Instead of header("Location:")
Principle: Ensure security when redirecting users.

Example:

wp_safe_redirect(home_url());
exit;

Bad Eaxmple:

header("Location: https://example.com");
exit;

Why:
 Prevents Open Redirect vulnerabilities.


16. Use Nonces for Security in Forms
Principle: Prevent CSRF attacks.

function my_form() {
    ?>
    <form method="post">
        <?php wp_nonce_field('my_nonce_action', 'my_nonce_field'); ?>
        <input type="submit" value="Submit">
    </form>
    <?php
}

function handle_form_submission() {
    if (!isset($_POST['my_nonce_field']) || !wp_verify_nonce($_POST['my_nonce_field'], 'my_nonce_action')) {
        die("Security check failed");
    }
}
add_action('admin_post_my_form', 'handle_form_submission');

Why:
Ensures form data is secure.

17. Avoid Global Variables (Use Dependency Injection)
Principle: Avoid polluting the global scope by using DI or proper encapsulation.


Example:

function get_users($wpdb) {
    return $wpdb->get_results("SELECT * FROM wp_users");
}

global $wpdb;
$users = get_users($wpdb);

Bad Example:

global $wpdb;
$results = $wpdb->get_results("SELECT * FROM wp_users");

Why:
This avoids directly modifying global variables.

18. Follow the WordPress Coding Standards


19. Always Use get_option() and update_option() for Settings

20. Use Custom Table Prefix for Security

21. Use Transients for Caching Data
22. Use sanitize_*() and esc_*() Functions
23. Use wp_nonce_field() and check_admin_referer() for Form Security

24. Never Modify WordPress Core Files
25. Use get_template_part() Instead of Hardcoding Templates

26. Use WP_Query Instead of Direct Database Queries

Good Example:

$query = new WP_Query([
    'post_type' => 'product',
    'posts_per_page' => 10,
]);

Bad Example:

global $wpdb;
$results = $wpdb->get_results("SELECT * FROM wp_posts WHERE post_type = 'product'");

27. Use wp_remote_get() Instead of file_get_contents()

28. Use wp_schedule_event() Instead of cron Jobs



29. Use Namespaces & Autoloading Instead of require or include


Good Example:

namespace MyPlugin;

class Product {
    public function get_name() {
        return "Sample Product";
    }
}

// Autoload using `composer.json`
"autoload": {
    "psr-4": {
        "MyPlugin\\": "inc/"
    }
}

Bad Example:

require 'inc/class-product.php';
$product = new Product();

Why:
Organizes code better and avoids manual require calls.

30. Avoid Hardcoded URLs, Use home_url() and site_url()

31. Optimize Database Queries with Indexing & WP_Query









32. Use WP_List_Table for Custom Admin Tables

Good Example:

if (!class_exists('WP_List_Table')) {
    require_once ABSPATH . 'wp-admin/includes/class-wp-list-table.php';
}

class My_Custom_Table extends WP_List_Table {
    function get_columns() {
        return [
            'cb'    => '<input type="checkbox" />',
            'title' => 'Title',
            'date'  => 'Date',
        ];
    }

    function prepare_items() {
        $this->_column_headers = [$this->get_columns(), [], []];
        $this->items = [
            ['ID' => 1, 'title' => 'Sample Item', 'date' => '2024-02-15'],
        ];
    }
}


33. Use register_block_type() for Custom Gutenberg Blocks Instead of Shortcodes

34. Use wp_kses() to Control Allowed HTML Tags

35. Avoid Using query_posts(), Use WP_Query Instead

Why:
Prevents conflicts with the main query and improves performance.

36. Use esc_attr(), esc_html(), and esc_url() for Output Sanitization

37. Use wp_localize_script() for Passing PHP Data to JavaScript

38. Use do_action() and apply_filters() to Make Code Extendable

Principle: Allow other developers to modify your code without editing core files.

Good Example:

function my_plugin_display_message() {
    $message = apply_filters('my_plugin_custom_message', 'Default Message');
    echo '<p>' . esc_html($message) . '</p>';
}
add_action('wp_footer', 'my_plugin_display_message');

// Another plugin can modify this message:
add_filter('my_plugin_custom_message', function ($message) {
    return 'This is a custom message!';
});

Why:
Makes your code modular and easily customizable.

39. Load Scripts Only Where Needed

40. Use wp_ajax_ & wp_ajax_nopriv_ for Secure AJAX Requests








```php
<?php
/*
Plugin Name: Email Button
Description: Adds a floating email button to the site with guaranteed email address passing.
Version: 1.1
Author: Sasha Zimin
Author URI: https://zimin.dev
License: GPL2
License URI: https://www.gnu.org/licenses/gpl-2.0.html
Text Domain: email-button-plugin
*/

if (!defined('ABSPATH')) exit;

// Admin settings
function eb_register_settings_menu() {
    add_options_page(
        'Email Button Settings',
        'Email Button',
        'manage_options',
        'email-button',
        'eb_settings_page'
    );
}
add_action('admin_menu', 'eb_register_settings_menu');

function eb_settings_page() {
    ?>
    <div class="wrap">
        <h1>Email Button Settings</h1>
        <form method="post" action="options.php">
            <?php settings_fields('eb_settings_group'); ?>
            <table class="form-table">
                <tr>
                    <th>Email Address</th>
                    <td>
                        <input type="email" name="eb_email_address" 
                               value="<?php echo esc_attr(get_option('eb_email_address')); ?>" 
                               required placeholder="contact@example.com">
                    </td>
                </tr>
                <tr>
                    <th>Default Subject</th>
                    <td>
                        <input type="text" name="eb_email_subject" 
                               value="<?php echo esc_attr(get_option('eb_email_subject')); ?>"
                               placeholder="Hello!">
                    </td>
                </tr>
                <tr>
                    <th>Button Position</th>
                    <td>
                        <select name="eb_button_position">
                            <option value="right" <?php selected(get_option('eb_button_position'), 'right'); ?>>Right</option>
                            <option value="left" <?php selected(get_option('eb_button_position'), 'left'); ?>>Left</option>
                        </select>
                    </td>
                </tr>
                <tr>
                    <th>Vertical Position</th>
                    <td>
                        <input type="number" name="eb_vertical_offset" 
                               value="<?php echo esc_attr(get_option('eb_vertical_offset', '25')); ?>"> px from bottom
                    </td>
                </tr>
                <tr>
                    <th>Button Color</th>
                    <td>
                        <input type="color" name="eb_button_color" 
                               value="<?php echo esc_attr(get_option('eb_button_color', '#4285F4')); ?>">
                    </td>
                </tr>
            </table>
            <?php submit_button(); ?>
        </form>
    </div>
    <?php
}

function eb_register_settings() {
    register_setting('eb_settings_group', 'eb_email_address', ['sanitize_callback' => 'sanitize_email']);
    register_setting('eb_settings_group', 'eb_email_subject');
    register_setting('eb_settings_group', 'eb_button_position', ['default' => 'right']);
    register_setting('eb_settings_group', 'eb_vertical_offset', ['default' => '25']);
    register_setting('eb_settings_group', 'eb_button_color', ['default' => '#4285F4']);
}
add_action('admin_init', 'eb_register_settings');

// Frontend button
function eb_add_email_button() {
    $email = get_option('eb_email_address');
    if (!$email) return;

    $subject = get_option('eb_email_subject') ?: 'Website Inquiry';
    $position = get_option('eb_button_position', 'right');
    $offset = get_option('eb_vertical_offset', '25');
    $color = get_option('eb_button_color', '#4285F4');
    
    $mailto = 'mailto:' . rawurlencode($email) . '?subject=' . rawurlencode($subject);
    
    echo '
    <a href="' . esc_url($mailto) . '" class="eb-float" title="Email Us">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="40" height="40" fill="#fff">
            <path d="M20 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2zm0 4l-8 5-8-5V6l8 5 8-5v2z"/>
        </svg>
    </a>
    <style>
        .eb-float {
            position: fixed;
            width: 70px;
            height: 70px;
            bottom: ' . (int)$offset . 'px;
            ' . ($position === 'left' ? 'left: 20px' : 'right: 20px') . ';
            background: ' . esc_attr($color) . ';
            border-radius: 50%;
            display: flex;
            align-items: center;// Prevent direct access
        )
    );
    
    register_setting(
        'eb_settings_group',
        'eb_email_subject'
    );
    
    register_setting(
        'eb_settings_group',
        'eb_button_position',
        array(
            'default' => 'right'
        )
    );
    
    register_setting(
        'eb_settings_group',
        'eb_vertical_offset',
        array(
            'default' => '25'
        )
    );
}
add_action('admin_init', 'eb_register_settings');

// Add Email button to frontend
function eb_add_email_button() {
    $email = get_option('eb_email_address');
    if (!$email) return;
    
    $subject = get_option('eb_email_subject');
    $position = get_option('eb_button_position', 'right');
    $offset = get_option('eb_vertical_offset', '25');
    
    // Build mailto link with proper encoding
    $mailto_args = array(
        'to' => $email,
        'subject' => $subject ? $subject : 'Website Inquiry'
    );
    
    $mailto_url = add_query_arg(array(
        'to' => rawurlencode($mailto_args['to']),
        'subject' => rawurlencode($mailto_args['subject'])
    ), 'mailto:');
    
    $position_css = ($position === 'left') ? 'left: 20px;' : 'right: 20px;';
    
    echo '
    <a href="' . esc_url($mailto_url) . '" class="eb-float" title="Send Email">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="40" height="40" fill="#fff">
            <path d="M20 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2zm0 4l-8 5-8-5V6l8 5 8-5v2z"/>
        </svg>
    </a>
    <style>
        .eb-float {
            position: fixed;
            width: 70px;
            height: 70px;
            bottom: ' . esc_attr($offset) . 'px;
            ' . $position_css . '
            background-color: #4285F4;
            color: #FFF;
            border-radius: 50px;
            box-shadow: 2px 2px 10px rgba(0,0,0,0.2);
            z-index: 1000;
            display: flex;
            justify-content: center;
            align-items: center;
            transition: all 0.3s ease;
        }
        .eb-float:hover {
            transform: scale(1.1);
            box-shadow: 3px 3px 15px rgba(0,0,0,0.3);
        }
        @media (max-width: 768px) {
            .eb-float {
                width: 60px;
                height: 60px;
            }
        }
    </style>
    ';
}
add_action('wp_footer', 'eb_add_email_button');
            justify-content: center;
            box-shadow: 0 2px 10px rgba(0,0,0,0.2);
            z-index: 999;
            transition: all 0.3s;
        }
        .eb-float:hover {
            transform: scale(1.1);
            box-shadow: 0 4px 15px rgba(0,0,0,0.3);
        }
        @media (max-width: 768px) {
            .eb-float {
                width: 60px;
                height: 60px;
            }
        }
    </style>
    ';
}
add_action('wp_footer', 'eb_add_email_button');
```

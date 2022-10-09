# wordpress-image-upload-api
This will help you upload image to wp rest api

#send a post request of your files to below method

```php

function uploadFile()
{
    require_once(ABSPATH . 'wp-admin/includes/image.php');
    require_once(ABSPATH . 'wp-admin/includes/file.php');
    require_once(ABSPATH . 'wp-admin/includes/media.php');
    //upload only images and files with the following extensions
    $file_extension_type = array('jpg', 'jpeg', 'jpe', 'gif', 'png', 'bmp', 'tiff', 'tif', 'ico', 'zip', 'pdf', 'docx');
    $file_extension = strtolower(pathinfo($_FILES['async-upload']['name'], PATHINFO_EXTENSION));
    if (!in_array($file_extension, $file_extension_type)) {
        return wp_send_json(
            array(
                'success' => false,
                'data'    => array(
                    'message'  => __('The uploaded file is not a valid file. Please try again.'),
                    'filename' => esc_html($_FILES['async-upload']['name']),
                ),
            )
        );
    }

    $attachment_id = media_handle_upload('async-upload', null, []);

    if (is_wp_error($attachment_id)) {
        return wp_send_json(
            array(
                'success' => false,
                'data'    => array(
                    'message'  => $attachment_id->get_error_message(),
                    'filename' => esc_html($_FILES['async-upload']['name']),
                ),
            )
        );
    }

    if (isset($post_data['context']) && isset($post_data['theme'])) {
        if ('custom-background' === $post_data['context']) {
            update_post_meta($attachment_id, '_wp_attachment_is_custom_background', $post_data['theme']);
        }

        if ('custom-header' === $post_data['context']) {
            update_post_meta($attachment_id, '_wp_attachment_is_custom_header', $post_data['theme']);
        }
    }

    $attachment = wp_prepare_attachment_for_js($attachment_id);
    if (!$attachment) {
        return wp_send_json(
            array(
                'success' => false,
                'data'    => array(
                    'message'  => __('Image cannot be uploaded.'),
                    'filename' => esc_html($_FILES['async-upload']['name']),
                ),
            )
        );
    }

    return wp_send_json(
        array(
            'success' => true,
            'data'    => $attachment,
        )
    );
}


```

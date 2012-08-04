<?php
/*
 * Plugin Name: Post Content Actions
 * Plugin URI: http://vinicius.soylocoporti.org.br/post-content-actions-wordpress-plugin
 * Description: Enable a set of shortcodes in the post content to set the post title, slug, categories, tags, custom fields and others. Useful for updating WordPress from platforms that don't fully support it, like e-mail and XMLRPC compatible apps.
 * Author: Vinicius Massuchetto
 * Version: 0.01
 * Author URI: http://vinicius.soylocoporti.org.br
*/

class PostContentActions {

    var $post;

    function PostContentActions () {
        add_action('save_post', array($this, 'save_post'));
    }

    function save_post($post_id) {
        $this->post = get_post($i = $post_id);
        $original_post = unserialize(serialize($this->post));

        $funcs = array(
            'title',
            'slug',
            'excerpt',
            'delay',
            'status',
            'password',
            'category',
            'tag',
            'custom_field',
            'add_custom_field',
            'comment',
            'ping'
        );
        foreach ($funcs as $f)
            call_user_func(array($this, $f));

        $update_post = false;

        if (strcmp($this->post->post_content, $original_post->post_content))
            $update_post = true;

        if ($update_post)
            wp_update_post($this->post);
    }

    function title() {
        $this->post->post_content = preg_replace_callback(
            '/\[title(.*)\]/mi',
            array($this, 'title_callback'),
            $this->post->post_content);
    }

    function title_callback ($matches) {
        $this->post->post_title = trim($matches[1]);
        return false;
    }

    function slug() {
        $this->post->post_content = preg_replace_callback(
            '/\[slug(.*)\]/mi',
            array($this, 'slug_callback'),
            $this->post->post_content);
    }

    function slug_callback ($matches) {
        $this->post->post_name = sanitize_title(trim($matches[1]));
        return false;
    }

    function excerpt() {
        $this->post->post_content = preg_replace_callback(
            '/\[excerpt\](.*)\[\/excerpt\]/mi',
            array($this, 'excerpt_callback'),
            $this->post->post_content);
    }

    function excerpt_callback($matches) {
        $this->post->post_excerpt = trim($matches[1]);
        return false;
    }

    function delay() {
        $this->post->post_content = preg_replace_callback(
            '/\[delay(.*)\]/mi',
            array($this, 'delay_callback'),
            $this->post->post_content);
    }

    function delay_callback ($matches) {
        $this->post->post_date = date('Y-m-d H:i:s', strtotime($matches[1]));
        return false;
    }

    function status() {
        $this->post->post_content = preg_replace_callback(
            '/\[status(.*)\]/mi',
            array($this, 'status_callback'),
            $this->post->post_content);
    }

    function status_callback ($matches) {
        $status = trim($matches[1]);
        $valid = array('publish', 'pending', 'draft', 'private');
        if (in_array($status, $valid))
            $this->post->post_status = $status;
        return false;
    }

    function password() {
        $this->post->post_content = preg_replace_callback(
            '/\[password(.*)\]/mi',
            array($this, 'password_callback'),
            $this->post->post_content);
    }

    function password_callback ($matches) {
        $this->post->post_password = trim($matches[1]);
        return false;
    }

    function category() {
        $this->post->post_content = preg_replace_callback(
            '/\[(category|categories)(.*)\]/mi',
            array($this, 'category_callback'),
            $this->post->post_content);
    }

    function category_callback($matches) {

        $categories = explode(',', $matches[2]);
        $insert_cat = array();

        foreach ($categories as $c) {
            if (!$cat_name = trim($c))
                continue;
            if (!$cat = get_term_by('name', $cat_name, 'category', ARRAY_A))
                $cat = wp_insert_term($cat_name, 'category');
            if (!is_wp_error($cat))
                $insert_cat[] = intval($cat['term_id']);
        }

        wp_set_object_terms($this->post->ID, $insert_cat, 'category');
        return false;
    }

    function tag() {
        $this->post->post_content = preg_replace_callback(
            '/\[(tags?)(.*)\]/mi',
            array($this, 'tag_callback'),
            $this->post->post_content);
    }

    function tag_callback($matches) {

        $tags = explode(',', $matches[2]);
        $insert_tag = array();

        foreach ($tags as $t) {
            if (!$tag_name = trim($t))
                continue;
            if (!$tag = get_term_by('name', $tag_name, 'post_tag', ARRAY_A))
                $tag = wp_insert_term($tag_name, 'post_tag');
            if (!is_wp_error($cat))
                $insert_tag[] = intval($tag['term_id']);
        }

        wp_set_object_terms($this->post->ID, $insert_tag, 'post_tag');
        return false;
    }


    function add_custom_field () {
        $this->post->post_content = preg_replace_callback(
            '/\[add\s*custom\s*field(.*),(.*)\]/mi',
            array($this, 'add_custom_field_callback'),
            $this->post->post_content);
    }

    function add_custom_field_callback ($matches) {
        add_post_meta($this->post->ID,
            trim($matches[1]), trim($matches[2]));
        return false;
    }

    function custom_field () {
        $this->post->post_content = preg_replace_callback(
            '/\[custom\s*field(.*),(.*)\]/mi',
            array($this, 'custom_field_callback'),
            $this->post->post_content);
    }

    function custom_field_callback ($matches) {
        update_post_meta($this->post->ID,
            trim($matches[1]), trim($matches[2]));
        return false;
    }

    function comment () {
        $this->post->post_content = preg_replace_callback(
            '/\[comments?(.*)\]/mi',
            array($this, 'comment_callback'),
            $this->post->post_content);
    }

    function comment_callback ($matches) {
        $param = trim($matches[1]);
        $valid_true = array('1', 'ok', 'on', 'enabled', 'open');
        $valid_false = array('0', 'no', 'off', 'disabled', 'closed');

        $comment_status = null;
        if (in_array($param, $valid_true))
            $comment_status = 'open';
        elseif (in_array($param, $valid_false))
            $comment_status = 'closed';

        if ($comment_status !== null)
            $this->post->comment_status = $comment_status;

        return false;
    }

    function ping () {
        $this->post->post_content = preg_replace_callback(
            '/\[ping(s|back|backs)?(.*)\]/mi',
            array($this, 'ping_callback'),
            $this->post->post_content);
    }

    function ping_callback ($matches) {
        $param = trim($matches[2]);
        $valid_true = array('1', 'ok', 'on', 'enabled', 'open');
        $valid_false = array('0', 'no', 'off', 'disabled', 'closed');

        $ping_status = null;
        if (in_array($param, $valid_true))
            $ping_status = 'open';
        elseif (in_array($param, $valid_false))
            $ping_status = 'closed';

        if ($ping_status !== null)
            $this->post->ping_status = $ping_status;

        return false;
    }

}

$pca = new PostContentActions();

?>

# Timmy

Timmy is an opt-in plugin for [Timber](http://upstatement.com/timber/) to make it even more convenient to work with images.

Timmy uses TimberImageHelper to handle images. This gives you the following advantages:

* **Image resizing on the fly**. Whenever a page is accessed and the image size can’t be found, it will be created on the fly. You can use as many different image sizes as you like, without always having to use plugins like [Regenerate Thumbnails](https://wordpress.org/plugins/regenerate-thumbnails/) when you make a change.
* **Advanced responsive image sizes**. For each image size, you can define additional sizes that will be used for the responsive image srcset. You will use this together with [Picturefill](https://scottjehl.github.io/picturefill/).
* **Restrict to Post Types**. If you want to use an image size just for one post type, you can define that.
* **User can select different image sizes in post**: Normally, a user could only select the sizes *Thumbnail*, *Medium*, *Large* and *Full*. With images defined through Timber a user can select all image sizes that you define.
* **You can still use Regenerate Thumbnails**. Say you need to edit your images for some reasons (for example if you need to change them from CMYK to RGB) and then upload them again. Just replace the full size images in the uploads folder and use [Regenerate Thumbnails](https://wordpress.org/plugins/regenerate-thumbnails/) to generate all other sizes. It will also clean your uploads folder from image sizes you don’t need anymore.

## Requirements

In order to make this work, you’ll have to

1. Install the [Timber Library Plugin](https://wordpress.org/plugins/timber-library/)
2. Set all Media Sizes in WordPress Settings to 0
    
    ![](http://i.imgur.com/CJlkO4Z.png)

3. In `functions.php`, make sure `set_post_thumbnail_size()` is also set to `0, 0`

    ```php
    set_post_thumbnail_size(0, 0);
    ```

4. Define a function `get_image_sizes()` in `functions.php` of your theme or in a file that is required or included from `functions.php`. This function will return an array with your [Image Configuration][image-configuration].

### Example

```php
function get_image_sizes() {
    return array(
        'custom-4' => array(
            'resize' => array( 370 ),
            'srcset' => array( 2 ),
            'size' => '(min-width: 992px) 33.333vw, 100vw',
            'name' => 'Width 1/4 fix',
            'post_types' => array( 'post', 'page' ),
        ),
    );
}
```

* The array key (*custom-4* in the example above) will be used to reference the image when you want to load it in your template.

## Functions

You can use the following functions to get your images into your template:

### get_timber_image()

`get_timber_image(int $post_id|TimberImage $timberImage, string $size)`

Returns the src attribute together with optional alt and title attributes for a TimberImage.


##### Usage in WordPress Templates

```php
<img<?php echo get_timber_image(get_post_thumbnail_id(), 'custom-4'); ?>>
```

##### Usage in Twig

For twig, this function is used as a filter on the TimberImage appended with a `|`.

```twig
<img{{post.thumbnail|get_timber_image('custom-4-crop')}}>
```

### get_timber_image_src()

`get_timber_image_src(int $post_id|TimberImage $timber_image, string $size)`

Returns the src for a TimberImage.

##### Usage in WordPress Templates

```php
<img src="<?php echo get_timber_image_src(get_post_thumbnail_id(), 'custom-4-crop'); ?>">
```

##### Usage in Twig

```twig
<img{{post.thumbnail|get_timber_image('custom-4-crop')}}>
```

### get_timber_image_responsive()

`get_timber_image_responsive(int $post_id|TimberImage $timber_image, string $size)`

Returns the srcset, size, alt and title attributes for a TimberImage.

##### Usage in WordPress Templates

```php
<img<?php echo get_timber_image_responsive(get_post_thumbnail_id(), 'custom-6'); ?>>
```

##### Usage in Twig

```twig
<img{{post.thumbnail|get_timber_image_responsive('custom-6')}}>
```

### get_timber_image_responsive_src()

Returns the srcset and sizes for a TimberImage. This is practically the same as `get_timber_image_responsive`, just without alt and title tags.

### get_timber_image_responsive_acf()

`get_timber_image_responsive_acf(string $field_name, string $size)`

Returns the same as `get_timber_image_responsive()`, but with the input of just an ACF image.

You can pass in the name of the ACF Field the image is saved in as the first parameter.

##### Usage in WordPress Templates

```php
<img<?php echo get_timber_image_responsive_acf('image', 'custom-4-crop'); ?>>
// will use get_field('image') to get the image information
```

##### Usage in Twig

You won’t use this function as a filter like the ones above.

```twig
<img{{get_timber_image_responsive_acf('image', 'custom-4-crop')}}>
```

### get_post_thumbnail()

`get_post_thumbnail(int $postId, string $size = 'post-thumbnail')`

Returns the src, alt and title attributes for a post thumbnail at a given size.

##### Usage in WordPress Templates

```php
<img<?php echo get_post_thumbnail(get_post_thumbnail_id(), 'custom-6'); ?>>
```

##### Usage in Twig

In Twig combined with Timber you will already have the post thumbnail through `post.thumbnail`. No need to wrap it in another function.

### get_post_thumbnail_src($postId, $size = 'post-thumbnail')

Returns the src for a post thumbnail. This is practically the same as `get_post_thumbnail`, just without alt and title tags.

## Image Configuration [image-configuration]

### resize `array()`

This is the normal size at which the image is displayed.

For each images size, you need to define a `resize` key that contains the parameters later given to the resize function (more about this on <https://github.com/jarednova/timber/wiki/Image-cookbook#arbitrary-resizing-of-images>).

```
'resize' => array( 370, 270 )
```

If you do not set a second value in the array, the image will not be cropped.

```
'resize' => array( 370 )
```

### srcset `array(array())` <small>(optional)</small>

These are alternative sizes for responsiveness. Read more about this on <http://scottjehl.github.io/picturefill/>.

For high-density screen support, you can add a bigger size than the standard image, provided the original upload image is at least that size. To save bandwith, it doesn’t necessarily have to be the doubled size, maybe a resize of 1.5 would be enough?

```
'srcset' => array(
    array( 768, 329 ),
    array( 480, 206 )
)
```

If you want, you can also use a ratio number of the size you want to use on the additional src. It will automatically scale the width and the height based on what is set in the 'resize' array.

```
'srcset' => array(
    0.3,
    0.5,
    2 // For a resize of ( 1400, 600 ), this is the same as array( 2800, 1200 )
)
```

### size `string` <small>(optional)</small>

This is the string for the sizes attribute for the picture polyfill. Read more about this on <http://scottjehl.github.io/picturefill/>.

```
/**
 * «For all screen widths above 62rem the image will be displayed at 33.333vw 
 * (33% of the viewport width), otherwise it will use 100vw (100% of the
 * viewport width).»
 */
'size' => '(min-width: 62rem) 33.333vw, 100vw'
```

```
/**
 * «For all screen widths up until 61.9375rem, the image will displayed at a
 * width of 125vm, for screen widths above at a width of 100vw.»
 */
'size' => '(max-width: 61.9375rem) 125vw, 100vw'
```

Picturefill will know which image size to use.

### post_types `array('', post', page)` <small>(optional)</small>

When you want to restrict image sizes to be only used for custom post types, you can define a `post_types` key containing an array with all the post types you want to allow. If you omit that key, post types `post` and `page as well as attachments not assigned to any post will be used as defaults.

Say you want an image sizes only to be used for Pages and the Employee post type:

```
'post_types' => array( 'page', 'employee' )
```

**all**

You can use `post_types' => array('all')` to always generate this size, for all post types.

### name `($string = '')` <small>(optional)</small>

The name parameter is used primarily for the backend. When the user chan choose an image size, e.g. when Media files are inserted into posts directly, a name would better describe what this image size is than only the key.

### show_in_ui `($bool = true)` <small>(optional)</small>

When you set this to false, the user will not be able to select that value in the backend when she e.g. wants to insert a Media file directly into the WYSYWIG content.

### generate_srcset_sizes `($bool = true)` <small>(optional)</small>

As per default, All the sizes defined under `srcset` will also be generated.

## Full Example

```
return array(
        'thumbnail' => array(
            'resize' => array( 150, 150 ),
            'name' => 'Vorschaubild',
            'post_types' => array('all'),
        ),
        'custom-4' => array(
            'resize' => array( 370 ),
            'size' => '(min-width: 62rem) 33.333vw, 100vw',
            'name' => 'Breite 1/4',
        ),
        'custom-4-crop' => array(
            'resize' => array( 370, 270 ),
            'srcset' => array( 2 ),
            'size' => '(min-width: 62rem) 33.333vw, 100vw',
            'name' => 'Breite 1/4 fix',
            'show_in_ui' => false,
            'post_types' => array( 'example', 'post', 'page' ),
        ),
        'custom-6' => array(
            // If you do not set a second value in the array, the image will not be cropped
            'resize' => array( 570 ),
            'srcset' => array( 0.5, 2 ),
            'size' => '(min-width: 62rem) 50vw, 100vw',
            'name' => 'Breite 1/2',
            'post_types' => array( 'example' ),
        ),
        // 14:6 crop
        'wide-normal' => array(
            // This is the normal size at which the image is displayed
            'resize' => array( 1400, 600 ),
            // These are alternative sizes for responsiveness
            'srcset' => array(
                array( 768, 329 ),
                array( 480, 206 ),
                2, // This is the same as array(2800, 1200)
            ),
            'size' => '(max-width: 61.9375rem) 125vw, 100vw',
            'name' => 'Volle Breite',
            'resize_srcset' => true,
        ),
    );

```

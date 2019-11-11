# Monolith WordPress Module

Monolith is a collection of utility functions and classes that make PHP and WordPress development a little bit easier. The WordPress module uses the `\Cgit\Monolith\WordPress` namespace.

## Install

Monolith WordPress is available on [Packagist](https://packagist.org/) and can be installed via [Composer](https://getcomposer.org/):

    composer require castlegate/monolith-wordpress

## Functions

*   `enqueue($file, $deps = [], $script = null, $parent = null)` Enqueues a CSS or JavaScript file with cache-busting, dependencies (based on handles), and automatic style or script detection. This function returns the automatically generated resource handle so you can use it in other dependency lists. Files can be specified by handle, relative path, absolute path, or URL.

*   `labels($single, $plural = null)` Generates a complete set of labels for a [custom post type](https://developer.wordpress.org/reference/functions/register_post_type/#parameters) or [custom taxonomy](https://developer.wordpress.org/reference/functions/register_taxonomy/#parameters).

*   `pagination($args = [])` Wrapper for `paginate_links()` with sensible default values.

*   `embedSvg($file, $title, $nofill)` Similar to the function of the same name in `Core` described above, but relative file paths are assumed to be in the active theme directory.

*   `send404()` Send an immediate 404 response and display the 404 template, overriding all other responses and output.

*   `pageFromTemplate($template, $multiple = false)` Identify and return the page(s) that use a particular template.

## Classes

### Post

Based on the original Terminus Post class, this provides convenient access to the final, filtered content of posts:

~~~ php
$foo = new \Cgit\Monolith\WordPress\Post(16);

echo $foo->id();
echo $foo->title();
echo $foo->url();
echo $foo->content();
echo $foo->excerpt();
~~~

It also provides the `field($field)` method, which retrieves an [Advanced Custom Fields](https://www.advancedcustomfields.com/) value from the post, and the `image()` method, which returns an `Image` instance for the post's featured image.

### Image

Based on the original Terminus Image class, this provides a consistent interface for getting URLs. Its constructor accepts an image attachment object, an image ID, a post object, a post ID, or an ACF custom field name.

~~~ php
use \Cgit\Monolith\WordPress\Image;

$foo = new Image($image_id); // image attachment
$foo = new Image($post_id); // featured image for post
$foo = new Image($field); // ACF image for current post
$foo = new Image($field, $post_id); // ACF image by post object or ID
~~~

You can then obtain various information about the image or generate an HTML `<img>` element:

~~~ php
$foo->url(); // URL of original image
$foo->url('medium'); // URL of image at a particular size
$foo->meta(); // get all meta information as an array
$foo->meta('alt'); // get particular meta field
$foo->element(); // get image element with fill size image
$foo->element('medium'); // get image element at a particular size
$foo->data('medium'); // get data URI
~~~

The `element()` method can also take an associative array of attribute keys and values to be added to the HTML element. If the `element()` method is provided with an associative array of sizes, it generates a responsive `<picture>` element:

~~~ php
$foo->element([
    'medium' => '(max-width: 480px)',
    'large' => '(max-width: 960px)',
]);
~~~

### PostType

The `PostType` class is an abstract class that can be extended to define custom post types with taxonomies, custom fields, and custom query parameters quickly and easily:

~~~ php
class Book extends PostType
{
    $this->type = 'book';

    $this->args = [
        'label' => 'Books',
        'label_single' => 'Book',
    ];

    $this->taxons = [
        'book-category' => [],
        'book-tag' => [],
    ];

    $this->queryArgs = [
        'posts_per_page' => 20,
    ];

    $this->fields = [
        [
            // ACF field group parameters
        ],
    ];
}
~~~

#### Properties

The `args` property is an array of `register_post_type` parameters, plus an optional `label_single` parameter that is used to generate a full set of labels automatically.

The `taxons` property is a nested array of taxonomy parameters. The keys are the taxonomy names and the values are the arrays of parameters passed to the `register_taxonomy` function.

The `queryArgs` property is an array of `WP_Query` parameters set via the `pre_get_posts` action.

The `fields` property is a nested array of ACF field groups, allowing you to add multiple to the post type. The `location` parameter can be omitted and will be set the current post type automatically.

#### Methods

You do not need to use any methods to define a post type. However, the class does call some methods can be extended in your child class, perhaps to set the post type parameters:

*   `init` is run immediately on instantiation.
*   `preInit` is run on the `init` action before any of the plugin methods have run.
*   `postInit` is run on the `init` action after all the plugin methods have run.

### Resource

The `Resource` class does the heavy lifting for the `enqueue` function and shouldn't need to be used directly.

~~~ php
$foo = new \Cgit\Monolith\WordPress\Resource($source, $deps);

$foo->enqueue(); // enqueue resource
$foo->setScript(true); // resource should be enqueued as a script
$foo->setParent(true); // resource path is relative to parent theme
$foo->getHandle(); // return generated resource handle
~~~

## License

Copyright (c) 2019 Castlegate IT. All rights reserved.

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.

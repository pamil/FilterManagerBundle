# Search page example

This example will be implemented on empty Symfony Standard project (can be previewed at [this](https://github.com/symfony/symfony-standard/tree/master) link).
Documents will be defined in already created `AppBundle`.

Make sure that you have ESB configured and working before continuing. More info about that in [official documentation](http://docs.ongr.io/ElasticsearchBundle).

## Sample data
In this example we will use `Product` documents:

```php
# src/AppBundle/Document/Product.php

namespace AppBundle\Document;

use ONGR\ElasticsearchBundle\Annotation as ES;

/**
 * @ES\Document
 */
class Product
{
    /**
     * @var string
     *
     * @ES\Property(type="keyword", options={"index"="not_analyzed"})
     */
    public $title;

    /**
     * @var string
     *
     * @ES\Property(type="keyword", options={"index"="not_analyzed"})
     */
    public $color;

    /**
     * @var string
     *
     * @ES\Property(type="keyword", options={"index"="not_analyzed"})
     */
    public $country;

    /**
     * @var string
     *
     * @ES\Property(type="float")
     */
    public $weight;

    /**
     * @var string
     *
     * @ES\Property(type="keyword", options={"index"=false})
     */
    public $image;

    /**
     * @var bool
     *
     * @ES\Property(type="boolean")
     */
    public $active;
}

```

## Define filters
Now filters have to be defined in configuration. This example defines single `search_list` manager and some filters:

```yaml
# app/config/config.yml

ongr_filter_manager:
    managers:
        search_list:
            filters:
                - search
                - color
                - country
                - weight
                - search_pager
                - search_sort
            repository: 'es.manager.default.product'
    filters:
        color:
            type: choice
            request_field: 'color'
            document_field: color
        country:
            type: multi_choice
            request_field: 'country'
            document_field: country
        search:
            type: match
            request_field: 'q'
            document_field: title
        weight:
            type: range
            request_field: 'weight'
            document_field: weight
        search_pager:
            type: pager
            document_field: ~
            request_field: 'page'
            options:
                count_per_page: 5
        only_active:
            type: field_value
            document_field: 'active'
            request_field: ~
            options:
                value: true
        search_sort:
            type: sort
            request_field: 'sort'
            document_field: ~
            options:
                choices:
                  - { label: No sorting, key: score, field: _score }
                  - { label: Heaviest to lightest, key: weight_desc, field: weight, order: desc }
                  - { label: Lightest to heaviest, key: weight_asc, field: weight, order: asc  }
```

## Define route

Next step is to define route for search page, let's add following lines to routing:
```yaml
# app/config/routing.yml

ongr_search_page:
    path: /search
    methods:  [GET]
    defaults:
        _controller: ONGRFilterManagerBundle:Manager:manager
        managerName: "search_list"
        template: "AppBundle::search.html.twig"
```

As seen from this example already predefined action `ONGRFilterManagerBundle:Manager:manager` will be used. We provide 
previously defined `search_list` manager. Search page will be reachable via `/search`.
Last parameter is template to use, see below for more information

## Templating

Our template will be placed in AppBundle's `Resources/views/search.html.twig` file. This template will get 
`filter_manager` variable from the inbuilt controller. It contains all information related to our filtered list.

> Please note that you can use your custom controller for this purpose. Check the [docs](http://docs.ongr.io/FilterManagerBundle/examples/custom_controller) how.

### Listing documents

Documents can be accessed through `filter_manager.result`. To make a dummy list of results put following code to your template:

```twig
{% for product in filter_manager.result %}
    <ul>
        <li>Title: {{ product.title }}</li>
        <li>Color: {{ product.color }}</li>
        <li>Country: {{ product.country }}</li>
        <li>Weight: {{ product.weight }}</li>
        <li>Image URL: {{ product.image }}</li>
        <li>Active: {{ product.active }}</li>
    </ul>
{% endfor %}
```

### Listing filters

Previously we assigned several filters to `search_list` filter manager. They are accessible via `filter_manager.filters`.
This returns the array with the `ViewData` objects associated with every filter. You can read more on that in 
[dedicated docs](http://docs.ongr.io/FilterManagerBundle/ViewData)

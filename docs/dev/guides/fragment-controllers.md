---
title: "Fragment Controllers"
description: "The concept behind fragment controllers."
aliases:
  - /guides/fragment-controllers
---


## Intro

A regular page in Contao consists of various partials that build the actual page content:

1. The page has a layout assigned, which renders the base template (usually `fe_page.html5`)
2. Each page layout has multiple layout sections (header, left, main, custom ones etc.)
3. Each layout section contain one or multiple modules or articles
4. Each article is a container filled with content elements

Splitting the page into these partials is the foundation of flexible page building in Contao.

{{% notice info %}}
The concept of modules and elements exists since the beginning of Contao.
Front end modules and content elements as Symfony controller fragments are available since **Contao 4.5**.
{{% /notice %}}

{{% notice warning %}}
Before you learn about fragments, make sure to
[understand Symfony controllers](https://symfony.com/doc/current/controller.html).
{{% /notice %}}


## Concept

In Symfony, an HTML template is usually generated by a controller. From the Symfony documentation:

> A controller is a PHP function you create that reads information from the Request object and
> creates and returns a Response object. The response could be an HTML page, JSON, XML, a
> file download, a redirect, a 404 error or anything else. The controller executes whatever
> arbitrary logic your application needs to render the content of a page.

A normal controller has a route – an URL on your site – that will generate the response.
As fragments are partials of an existing page; they do not have a route, but same as a
normal controller they will get a request and generate a response. This concept is called
[*Sub Requests* in Symfony][subrequests], each fragment in Contao is a sub request in Symfony.

From an HTTP perspective, each (sub) request is totally independent. Each one will get request
parameters (like `GET` and `POST` data), and they can return HTTP headers (like `Content Type`
or `Cache-Control`).

{{% notice info %}}
{{< version-tag "4.9" >}} Subrequests can generate [a response that modifies the cache time of
the main response](/framework/caching/#caching-fragments).
{{% /notice %}}



## Fragment Controllers

Contao provides two different types of fragments out-of-the-box:
[Front End Modules][modules] and [Content Elements][elements]. For each of
these, there are abstract classes (namely `AbstractFrontendModuleController`
and `AbstractContentElementController`) that mimick the behavior of a Contao
module/element. They prepare the Contao template based on the request and current
model, and allow to return a response with the fragment result.


### Controllers in the Contao Framework

Front end modules and content elements are still generated by the Contao
framework and registered in the `$GLOBALS['FE_MOD']` and `$GLOBALS['TL_CTE']`
respectively. The `ModuleProxy` and `ContentProxy` classes are responsible
for making a fragment controller behave like a Contao module/element.


### Extending Legacy Classes

{{% notice tip %}}
Modules are used to simplify, everything mentioned here works exactly the same for content elements.
{{% /notice %}}

Sometimes, one might need to override/extend the functionality of an existing Contao module.
While you can still create a module the legacy way, it limits functionality and is not
recommended anymore. However, there's nothing limiting us from extending an existing class and
using it as a controller in Contao 4!

```php
// src/Controller/FrontendModule/AppExampleController.php
namespace App\Controller\FrontendModule;

use Contao\CoreBundle\DependencyInjection\Attribute\AsFrontendModule;
use Contao\ModuleModel;
use Contao\ModuleNewsList;
use Symfony\Component\HttpFoundation\Response;

#[AsFrontendModule(category: 'news', priority: 1)]
class AppExampleController extends ModuleNewsList
{
    public function __construct() {}

    public function __invoke(ModuleModel $model, string $section): Response
    {
        parent::__construct($model, $section);

        return new Response($this->generate());
    }
    
    // Do whatever you want here, e.g. override parent methods
}
```

{{%expand "Code example to extend a Contao content element" %}}
```php
// src/Controller/ContentElement/AppExampleController.php
namespace App\Controller\ContentElement;

use Contao\ContentGallery;
use Contao\ContentModel;
use Contao\CoreBundle\DependencyInjection\Attribute\AsContentElement;
use Symfony\Component\HttpFoundation\Response;

#[AsContentElement(category: 'media', priority: 1)]
class AppExampleController extends ContentGallery
{
    public function __construct() {}

    public function __invoke(ContentModel $model, string $section): Response
    {
        parent::__construct($model, $section);

        return new Response($this->generate());
    }
    
    // Do whatever you want here, e.g. override parent methods
}
```
{{% /expand%}}

{{% notice "info" %}}
Since we are extending classes from the legacy Contao framework here, these controllers will not be automatically registered as a service
by Contao. Therefore you will need to specifically register these controllers as services in your own `config/services.yaml`. See 
[this article](/getting-started/starting-development#autoloading-services-and-actions) for more information.
{{% /notice %}}


## Adding custom fragment types

The Contao fragment registry is able to render _any_ fragment, not just frontend modules
and content elements. As an example, a shop extension could support fragments for different
product types, allowing third-party extension to add new product types to the shop system.
To fully understand the concept, you might need strong Symfony knowledge and to reverse-engineer
the Contao core. Make sure to have a look at the `FragmentRegistry` and `RegisterFragmentsPass`.



## Read more

* [Front End Modules][modules]
* [Content Elements][elements]


[modules]: /framework/front-end-modules/
[elements]: /framework/content-elements/
[subrequests]: https://symfony.com/doc/current/components/http_kernel.html#sub-requests

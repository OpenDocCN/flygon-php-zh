# 第六章。构建核心模块

到目前为止，我们已经熟悉了 PHP 7 的最新变化，设计模式，设计原则和流行的 PHP 框架。我们还更详细地了解了 Symfony 作为我们未来的框架选择。现在我们终于达到了一个可以开始构建我们的模块化应用程序的地步。使用 Symfony 构建模块化应用程序是通过 bundles 机制完成的。从术语上讲，从这一点开始，我们将考虑 bundle 和模块是相同的东西。

在本章中，我们将涵盖以下与核心模块相关的主题：

+   要求

+   依赖关系

+   实施

+   单元测试

+   功能测试

# 要求

回顾第四章, *模块化网络商店应用的需求规范*，以及那里提出的线框图，我们可以概述这个模块将具有的一些要求。核心模块将用于设置通用的、应用程序范围的功能，如下：

+   将 Foundation CSS for sites 包含到项目中

+   构建主页

+   构建其他静态页面

+   构建一个联系我们页面

+   设置基本防火墙，其中管理员用户可以管理稍后其他模块生成的 CRUD

# 依赖关系

核心模块本身并不依赖于我们将作为本书一部分编写的其他模块，或者 Symfony 标准安装之外的任何第三方模块。

# 实施

我们首先创建一个全新的 Symfony 项目，运行以下控制台命令：

```php
**symfony new shop**

```

这将创建一个新的`shop`目录，其中包含在浏览器中运行我们的应用程序所需的所有文件。在这些文件和目录中，有一个`src/AppBundle`目录，实际上就是我们的核心模块。在我们可以在浏览器中运行应用程序之前，我们需要将新创建的`shop`目录映射到一个主机名，比如说`shop.app`，这样我们就可以通过`http://shop.app` URL 在浏览器中访问它。完成这一步后，如果我们打开`http://shop.app`，我们应该看到**欢迎来到 Symfony 3.1.0**的屏幕，如下所示：

![实现](img/B05460_06_05.jpg)

虽然我们目前还没有数据库的需求，但我们稍后将开发的其他模块将假定数据库连接，因此从一开始就进行设置是值得的。我们通过配置`app/config/parameters.yml`文件来配置正确的数据库连接参数。

然后我们从[`foundation.zurb.com/sites.html`](http://foundation.zurb.com/sites.html)下载 Foundation for Sites。下载完成后，我们需要解压并将`/js`和`/css`目录复制到`Symfony /web`目录中，如下面的屏幕截图所示：

![实现](img/B05460_06_06.jpg)

### 注意

值得注意的是，我们在模块中使用的 Foundation 是一个简化的设置，我们只是使用 CSS 和 JavaScript 文件，而没有设置任何与 Sass 相关的内容。

在 Foundation CSS 和 JavaScript 文件就位后，我们编辑`app/Resources/views/base.html.twig`文件如下：

```php
<!doctype html>
<html class="no-js"lang="en">
  <head>
    <meta charset="utf-8"/>
    <meta http-equiv="x-ua-compatible" content="ie=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>{% block title %}Welcome!{% endblock %}</title>
    <link rel="stylesheet"href="{{ asset('css/foundation.css') }}"/>
    {% block stylesheets%}{% endblock %}
  </head>
  <body>
    <!-- START BODY -->
    <!-- TOP-MENU -->
    <!-- SYSTEM-WIDE-MESSAGES -->
    <!-- PER-PAGE-BODY -->
    <!-- FOOTER -->
    <!-- START BODY -->
    <script src="{{ asset('js/vendor/jquery.js') }}"></script>
    <script src="{{ asset('js/vendor/what-input.js') }}"></script>
    <script src="{{ asset('js/vendor/foundation.js') }}"></script>
    <script>
      $(document).foundation();
    </script>
    {% block javascripts%}{% endblock %}
  </body>
</html>
```

在这里，我们设置整个头部和 body 结束区域，以及所有必要的 CSS 和 JavaScript 加载。Twig 的`asset`标签帮助我们构建 URL 路径，我们只需传递 URL 路径本身，它就会为我们构建一个完整的 URL。关于页面的实际内容，这里有几件事情需要考虑。我们将如何构建类别、客户和结账菜单？在这一点上，我们还没有这些模块，我们也不想让它们成为核心模块的必需品。那么我们如何解决还不存在的东西的挑战呢？

对于类别、客户和结账菜单，我们可以为每个菜单项定义全局 Twig 变量，然后用这些变量来渲染菜单。这些变量将通过适当的服务进行填充。由于核心包不知道未来的目录、客户和结账模块，我们将首先创建一些虚拟服务，并将它们连接到全局 Twig 变量。稍后，当我们开发目录、客户和结账模块时，这些模块将覆盖适当的服务，从而为菜单提供正确的值。

这种方法可能不完全符合模块化应用程序的概念，但对我们的需求来说已经足够了，因为我们并没有硬编码任何依赖关系。

我们首先在`app/config/config.yml`文件中添加以下条目：

```php
twig:
# ...
globals:
category_menu: '@category_menu'
customer_menu: '@customer_menu'
checkout_menu: '@checkout_menu'
products_bestsellers: '@bestsellers'
products_onsale: '@onsale'
```

`category_menu_items`、`customer_menu_items`、`checkout_menu_items`、`products_bestsellers`和`products_onsale`变量成为全局 Twig 变量，我们可以在任何 Twig 模板中使用，如下例所示：

```php
<ul>
  {% for category in category_menu.getItems() %}
  <li>{{ category.name }}</li>
  {% endfor %}
</ul>
```

Twig 全局变量`config`中的`@`字符用于表示服务名称的开始。这是将为我们的 Twig 变量提供值对象的服务。接下来，我们继续修改`app/config/services.yml`，创建`category_menu`、`customer_menu`、`checkout_menu`、`bestsellers`和`onsale`服务：

```php
services:
category_menu:
  class: AppBundle\Service\Menu\Category
customer_menu:
  class: AppBundle\Service\Menu\Customer
checkout_menu:
  class: AppBundle\Service\Menu\Checkout
bestsellers:
  class: AppBundle\Service\Menu\BestSellers
onsale:
  class: AppBundle\Service\Menu\OnSale
```

此外，我们在`src/AppBundle/Service/Menu/`目录下创建列出的每个服务类。我们首先从`src/AppBundle/Service/Menu/Bestsellers.php`文件开始，内容如下：

```php
namespace AppBundle\Service\Menu;

class BestSellers {
  public function getItems() {
    // Note, this can be arranged as per some "Product"interface, so to know what dummy data to return
    return array(
      ay('path' =>'iphone', 'name' =>'iPhone', 'img' =>'/img/missing-image.png', 'price' => 49.99, 'add_to_cart_url' =>'#'),
      array('path' =>'lg', 'name' =>'LG', 'img' =>
        '/img/missing-image.png', 'price' => 19.99, 'add_to_cart_url' =>'#'),
      array('path' =>'samsung', 'name' =>'Samsung', 'img'=>'/img/missing-image.png', 'price' => 29.99, 'add_to_cart_url' =>'#'),
      array('path' =>'lumia', 'name' =>'Lumia', 'img' =>'/img/missing-image.png', 'price' => 19.99, 'add_to_cart_url' =>'#'),
      array('path' =>'edge', 'name' =>'Edge', 'img' =>'/img/missing-image.png', 'price' => 39.99, 'add_to_cart_url' =>'#'),
    );
  }
}
```

然后，我们添加`src/AppBundle/Service/Menu/Category.php`文件，内容如下：

```php
class Category {
  public function getItems() {
    return array(
      array('path' =>'women', 'label' =>'Women'),
      array('path' =>'men', 'label' =>'Men'),
      array('path' =>'sport', 'label' =>'Sport'),
    );
  }
}
```

接下来，我们添加`src/AppBundle/Service/Menu/Checkout.php`文件，内容如下所示：

```php
class Checkout
{
  public function getItems()
  {
     // Initial dummy menu
     return array(
       array('path' =>'cart', 'label' =>'Cart (3)'),
       array('path' =>'checkout', 'label' =>'Checkout'),
    );
  }
}
```

完成后，我们将继续向`src/AppBundle/Service/Menu/Customer.php`文件添加以下内容：

```php
class Customer
{
  public function getItems()
  {
    // Initial dummy menu
    return array(
      array('path' =>'account', 'label' =>'John Doe'),
      array('path' =>'logout', 'label' =>'Logout'),
    );
  }
}
```

然后我们添加`src/AppBundle/Service/Menu/OnSale.php`文件，内容如下：

```php
class OnSale
{
  public function getItems()
  {
    // Note, this can be arranged as per some "Product" interface, so to know what dummy data to return
    return array(
      array('path' =>'iphone', 'name' =>'iPhone', 'img' =>'/img/missing-image.png', 'price' => 19.99, 'add_to_cart_url' =>'#'),
      array('path' =>'lg', 'name' =>'LG', 'img' =>'/img/missing-image.png', 'price'      => 29.99, 'add_to_cart_url' =>'#'),
      array('path' =>'samsung', 'name' =>'Samsung', 'img'=>'/img/missing-image.png', 'price' => 39.99, 'add_to_cart_url' =>'#'),
      array('path' =>'lumia', 'name' =>'Lumia', 'img' =>'/img/missing-image.png', 'price' => 49.99, 'add_to_cart_url' =>'#'),
      array('path' =>'edge', 'name' =>'Edge', 'img' =>'/img/missing-image.png', 'price' => 69.99, 'add_to_cart_url' =>'#'),
    ;
  }
}
```

我们现在已经定义了五个全局 Twig 变量，将用于构建我们的应用程序菜单。尽管变量现在连接到一个返回的虚拟数组的虚拟服务，但我们已经有效地将菜单项解耦到其他即将构建的模块中。当我们稍后开始构建我们的目录、客户和结账模块时，我们只需编写一个服务覆盖，并使用真实的项目填充菜单项数组。这将是理想的情况。

### 注意

理想情况下，我们希望我们的服务按照某种接口返回数据，以确保谁覆盖或扩展它都是通过接口来实现的。由于我们试图保持我们的应用程序最小化，我们将继续使用简单的数组。

现在我们可以回到我们的`app/Resources/views/base.html.twig`文件，用以下内容替换前面代码中的`<!-- TOP-MENU -->`：

```php
<div class="title-bar" data-responsive-toggle="appMenu" data-hide-for="medium">
  <button class="menu-icon" type="button" data-toggle></button>
  <div class="title-bar-title">Menu</div>
</div>

<div class="top-bar" id="appMenu">
  <div class="top-bar-left">
    {# category_menu is global twig var filled from service, and later overriden by another module service #}
    <ul class="menu">
      <li><a href="{{ path('homepage') }}">HOME</a></li>
        {% block category_menu %}
        {% for link in category_menu.getItems() %}
      <li><a href="{{ link.path }}">{{ link.label }}</li></a>
      {% endfor %}
      {% endblock %}
    </ul>
  </div>
  <div class="top-bar-right">
    <ul class="menu">
      {# customer_menu is global twig var filled from service, and later overriden by another module service #}
      {% block customer_menu %}
      {% for link in customer_menu.getItems() %}
      <li><a href="{{ link.path }}">{{ link.label }}</li></a>
      {% endfor %}
      {% endblock %}
      {# checkout_menu is global twig var filled from service, and later overriden by another module service #}
      {% block checkout_menu %}
      {% for link in checkout_menu.getItems() %}
      <li><a href="{{ link.path }}">{{ link.label }}</li></a>
      {% endfor %}
      {% endblock %}
    </ul>
  </div>
</div>
```

然后我们用以下内容替换`<!-- SYSTEM-WIDE-MESSAGES -->`：

```php
<div class="row column">
  {% for flash_message in app.session.flashBag.get('alert') %}
  <div class="alert callout">
    {{ flash_message }}
  </div>
  {% endfor %}
  {% for flash_message in app.session.flashBag.get('warning') %}
  <div class="warning callout">
    {{ flash_message }}
  </div>
  {% endfor %}
  {% for flash_message in app.session.flashBag.get('success') %}
  <div class="success callout">
    {{ flash_message }}
  </div>
  {% endfor %}
</div>
```

我们用以下内容替换`<!-- PER-PAGE-BODY -->`：

```php
<div class="row column">
  {% block body %}{% endblock %}
</div>
```

我们用以下内容替换`<!-- FOOTER -->`：

```php
<div class="row column">
  <ul class="menu">
    <li><a href="{{ path('about') }}">About Us</a></li>
    <li><a href="{{ path('customer_service') }}">Customer Service</a></li>
    <li><a href="{{ path('privacy_cookie') }}">Privacy and Cookie Policy</a></li>
    <li><a href="{{ path('orders_returns') }}">Orders and Returns</a></li>
    <li><a href="{{ path('contact') }}">Contact Us</a></li>
  </ul>
</div>
```

现在我们可以继续编辑`src/AppBundle/Controller/DefaultController.php`文件，并添加以下代码：

```php
/**
 * @Route("/", name="homepage")
 */
public function indexAction(Request $request)
{
  return $this->render('AppBundle:default:index.html.twig');
}

/**
 * @Route("/about", name="about")
 */
public function aboutAction()
{
  return $this->render('AppBundle:default:about.html.twig');
}

/**
 * @Route("/customer-service", name="customer_service")
 */
public function customerServiceAction()
{
  return $this->render('AppBundle:default:customer-service.html.twig');
}

/**
 * @Route("/orders-and-returns", name="orders_returns")
 */
public function ordersAndReturnsAction()
{
  return $this->render('AppBundle:default:orders-returns.html.twig');
}

/**
 * @Route("/privacy-and-cookie-policy", name="privacy_cookie")
 */
public function privacyAndCookiePolicyAction()
{
  return $this->render('AppBundle:default:privacy-cookie.html.twig');
}
```

位于`src/AppBundle/Resources/views/default`目录中的所有使用的模板文件（`about.html.twig`、`customer-service.html.twig`、`orders-returns.html.twig`、`privacy-cookie.html.twig`）可以类似地定义如下：

```php
{% extends 'base.html.twig' %}

{% block body %}
<div class="row">
  <h1>About Us</h1>
</div>
<div class="row">
  <p>Loremipsum dolor sit amet, consecteturadipiscingelit...</p>
</div>
{% endblock %}
```

在这里，我们只是将标题和内容包装到带有`row`类的`div`元素中，以便给它一些结构。结果应该是类似于这里显示的页面：

![实现](img/B05460_06_04.jpg)

**联系我们**页面需要不同的方法，因为它将包含一个表单。为了构建一个表单，我们使用 Symfony 的`Form`组件，通过向`src/AppBundle/Controller/DefaultController.php`文件添加以下内容：

```php
/**
 * @Route("/contact", name="contact")
 */
public function contactAction(Request $request) {

  // Build a form, with validation rules in place
  $form = $this->createFormBuilder()
  ->add('name', TextType::class, array(
    'constraints' => new NotBlank()
  ))
  ->add('email', EmailType::class, array(
    'constraints' => new Email()
  ))
  ->add('message', TextareaType::class, array(
    'constraints' => new Length(array('min' => 3))
  ))
   ->add('save', SubmitType::class, array(
    'label' =>'Reach Out!',
    'attr' => array('class' =>'button'),
  ))
  ->getForm();

  // Check if this is a POST type request and if so, handle form
  if ($request->isMethod('POST')) {
    $form->handleRequest($request);

    if ($form->isSubmitted() && $form->isValid()) {
      $this->addFlash(
        'success',
        'Your form has been submitted. Thank you.'
      );

      // todo: Send an email out...

      return $this->redirect($this->generateUrl('contact'));
    }
  }

  // Render "contact us" page
  return $this->render('AppBundle:default:contact.html.twig', array(
    'form' => $form->createView()
  ));
}
```

我们首先通过表单构建器构建了一个表单。`add`方法接受字段定义和字段约束，验证可以基于它们进行。然后我们添加了对 HTTP POST 方法的检查，如果是这种情况，我们将使用请求参数填充表单并对其进行验证。

通过`contactAction`方法，我们仍然需要一个模板文件来实际渲染表单。我们通过添加`src/AppBundle/Resources/views/default/contact.html.twig`文件并添加以下内容来实现：

```php
{% extends 'base.html.twig' %}

{% block body %}

<div class="row">
  <h1>Contact Us</h1>
</div>

<div class="row">
  {{ form_start(form) }}
  {{ form_widget(form) }}
  {{ form_end(form) }}
</div>
{% endblock %}
```

根据这些标签，Twig 为我们处理了表单渲染。结果的浏览器输出如下所示：

![实现](img/B05460_06_03.jpg)

我们几乎已经准备好所有的页面了。但是还有一件事缺失，就是我们主页的正文区域。与其他具有静态内容的页面不同，这个页面实际上是动态的，因为它列出了畅销书和特价商品。这些数据预计来自其他模块，目前还不可用。但是，这并不意味着我们不能为它们准备虚拟占位符。让我们继续编辑`app/Resources/views/default/index.html.twig`文件如下：

```php
{% extends 'base.html.twig' %}
{% block body %}
<!--products_bestsellers -->
<!--products_onsale -->
{% endblock %}
```

现在我们需要用以下内容替换`<!-- products_bestsellers -->`：

```php
{% if products_bestsellers %}
<h2 class="text-center">Best Sellers</h2>
<div class="row products_bestsellers text-center small-up-1 medium-up-3 large-up-5" data-equalizer data-equalize-by- row="true">
  {% for product in products_bestsellers.getItems() %}
  <div class="column product">
    <img src="{{ asset(product.img) }}" alt="missing image"/>
    <a href="{{ product.path }}">{{ product.name }}</a>
    <div>${{ product.price }}</div>
    <div><a class="small button"href="{{ product.add_to_cart_url }}">Add to Cart</a></div>
  </div>
  {% endfor %}
</div>
{% endif %}
```

现在我们需要用以下内容替换`<!-- products_onsale -->`：

```php
{% if products_onsale %}
<h2 class="text-center">On Sale</h2>
<div class="row products_onsale text-center small-up-1 medium-up-3 large-up-5" data-equalizer data-equalize-by-row="true">
  {% for product in products_onsale.getItems() %}
  <div class="column product">
    <img src="{{ asset(product.img) }}" alt="missing image"/>
    <a href="{{ product.path }}">{{ product.name }}</a>
  <div>${{ product.price }}</div>
  <div><a class="small button"href="{{ product.add_to_cart_url }}">Add to Cart</a></div>
  </div>
  {% endfor %}
</div>
{% endif %}
```

### 提示

[`dummyimage.com`](http://dummyimage.com)使我们能够为我们的应用程序创建占位图像。

此时，我们应该看到如下所示的主页：

![实现](img/B05460_06_02.jpg)

## 配置应用程序范围的安全性

作为应用程序范围安全的一部分，我们试图设置一些基本保护，防止未来的客户或任何其他用户能够访问和使用未来自动生成的 CRUD 控制器。我们通过修改`app/config/security.yml`文件来实现这一点。`security.yml`文件有几个组件需要处理：防火墙、访问控制、提供程序和编码器。如果我们观察先前测试应用程序中自动生成的 CRUD，就会清楚地看到我们需要保护以下内容，以防止客户访问：

+   `GET|POST /new`

+   `GET|POST /{id}/edit`

+   `DELETE /{id}`

换句话说，所有在 URL 中包含`/new`和`/edit`，以及所有使用`DELETE`方法的内容，都需要受到客户的保护。考虑到这一点，我们将使用 Symfony 安全功能创建一个具有`ROLE_ADMIN`角色的内存用户。然后，我们将创建一个访问控制列表，只允许`ROLE_ADMIN`访问我们刚刚提到的资源，并创建一个防火墙，当我们尝试访问这些资源时触发 HTTP 基本身份验证登录表单。

使用内存提供程序意味着在我们的`security.yml`文件中硬编码用户。对于我们应用程序的目的，我们将为管理员类型的用户这样做。然而，实际密码不需要硬编码。假设我们将使用`1L6lllW9zXg0`作为密码，让我们跳转到控制台并输入以下命令：

```php
**php bin/console security:encode-password**

```

这将产生以下输出。

![配置应用程序范围的安全性](img/B05460_06_01.jpg)

我们现在可以通过添加内存提供程序并将生成的编码密码复制粘贴到其中来编辑`security.yml`，如下所示：

```php
security:
    providers:
        in_memory:
            memory:
                users:
                    john:
                        password: $2y$12$DFozWehwPkp14sVXr7.IbusW8ugvmZs9dQMExlggtyEa/TxZUStnO
                        roles: 'ROLE_ADMIN'
```

在这里，我们定义了一个具有`ROLE_ADMIN`角色和编码`1L6lllW9zXg0`密码的用户`john`。

一旦我们有了提供程序，我们就可以继续在`security.yml`文件中添加编码器。否则 Symfony 将不知道如何处理分配给`john`用户的当前密码：

```php
security:
    encoders:
        Symfony\Component\Security\Core\User\User:
            algorithm: bcrypt
            cost: 12
```

然后我们添加防火墙如下：

```php
security:
    firewalls:
        guard_new_edit:
            pattern: /(new)|(edit)
            methods: [GET, POST]
            anonymous: ~
            http_basic: ~
       guard_delete:
           pattern: /
           methods: [DELETE]
           anonymous: ~
           http_basic: ~
```

`guard_new_edit`和`guard_delete`是我们两个应用程序防火墙的自由名称。`guard_new_edit`防火墙将拦截包含`/new`或`/edit`字符串的任何路由的所有 GET 和 POST 请求。`guard_delete`防火墙将拦截任何 URL 上的任何 HTTP `DELETE`方法。一旦这些防火墙启动，它们将显示一个 HTTP 基本身份验证表单，并且只有在用户登录后才允许访问。

然后我们按以下方式添加访问控制列表：

```php
security:
    access_control:
      # protect any possible auto-generated CRUD actions from everyone's access
      - { path: /new, roles: ROLE_ADMIN }
      - { path: /edit, roles: ROLE_ADMIN }
      - { path: /, roles: ROLE_ADMIN, methods: [DELETE] }
```

有了这些条目，任何试图访问任何 URL 的人，只要符合`access_control`下定义的任何模式，都将看到浏览器登录，如下所示：

![配置应用程序范围的安全性](img/B05460_06_07.jpg)

唯一可以登录的用户是`john`，密码是`1L6lllW9zXg0`。一旦认证，用户可以访问所有的 CRUD 链接。这对于我们简单的应用程序应该足够了。

# 单元测试

我们当前的模块除了控制器类和虚拟服务类之外没有特定的类。因此，我们不会在这里费心进行单元测试。

# 功能测试

在我们开始编写功能测试之前，我们需要通过将我们的 bundle `Tests`目录添加到`testsuite`路径中来编辑`phpunit.xml.dist`文件，如下所示：

```php
<testsuites>
  <testsuite name="Project Test Suite">
    <-- ... other elements ... -->
      <directory>src/AppBundle/Tests</directory>
    <-- ... other elements ... -->
  </testsuite>
</testsuites>
```

我们的功能测试将只覆盖一个控制器，因为我们没有其他控制器。我们首先创建一个`src/AppBundle/Tests/Controller/DefaultControllerTest.php`文件，内容如下：

```php
namespace AppBundle\Tests\Controller;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class DefaultControllerTest extends WebTestCase
{
//…
}
```

下一步是测试我们的每一个控制器动作。至少我们应该测试页面内容是否被正确输出。

### 提示

为了在我们的 IDE 中获得自动完成，我们可以从官方网站[`phpunit.de`](https://phpunit.de)下载`PHPUnitphar`文件。下载后，我们可以简单地将其添加到项目的根目录，这样 IDE（如**PHPStorm**）就可以识别它。这样就可以轻松地跟踪所有`$this->assert`方法调用及其参数。

我们想要测试的第一件事是我们的主页。我们通过向`DefaultControllerTest`类的主体添加以下内容来实现这一点。

```php
public function testHomepage()
{
  // @var \Symfony\Bundle\FrameworkBundle\Client
  $client = static::createClient();
  /** @var \Symfony\Component\DomCrawler\Crawler */
  $crawler = $client->request('GET', '/');

  // Check if homepage loads OK
  $this->assertEquals(200, $client->getResponse()->getStatusCode());

  // Check if top bar left menu is present
  $this->assertNotEmpty($crawler->filter('.top-bar-left li')->count());

  // Check if top bar right menu is present
  $this->assertNotEmpty($crawler->filter('.top-bar-right li')->count());

  // Check if footer is present
  $this->assertNotEmpty($crawler->filter('.footer li')->children()->count());
}
```

在这里，我们一次检查了几件事。我们检查页面是否正常加载，HTTP 200 状态。然后我们抓取左右菜单并计算它们的项目数，以查看它们是否有任何项目。如果所有单独的检查都通过了，`testHomepage`测试就被认为是通过的。

我们通过向`DefaultControllerTest`类添加以下内容来进一步测试所有静态页面：

```php
public function testStaticPages()
{
  // @var \Symfony\Bundle\FrameworkBundle\Client
  $client = static::createClient();
  /** @var \Symfony\Component\DomCrawler\Crawler */

  // Test About Us page
  $crawler = $client->request('GET', '/about');
  $this->assertEquals(200, $client->getResponse()->getStatusCode());
  $this->assertContains('About Us', $crawler->filter('h1')->text());

  // Test Customer Service page
  $crawler = $client->request('GET', '/customer-service');
  $this->assertEquals(200, $client->getResponse()->getStatusCode());
  $this->assertContains('Customer Service', $crawler->filter('h1')->text());

  // Test Privacy and Cookie Policy page
  $crawler = $client->request('GET', '/privacy-and-cookie-policy');
  $this->assertEquals(200, $client->getResponse()->getStatusCode());
  $this->assertContains('Privacy and Cookie Policy', $crawler->filter('h1')->text());

  // Test Orders and Returns page
  $crawler = $client->request('GET', '/orders-and-returns');
  $this->assertEquals(200, $client->getResponse()->getStatusCode());
  $this->assertContains('Orders and Returns', $crawler->filter('h1')->text());

  // Test Contact Us page
  $crawler = $client->request('GET', '/contact');
  $this->assertEquals(200, $client->getResponse()->getStatusCode());
  $this->assertContains('Contact Us', $crawler->filter('h1')->text());
}
```

在这里，我们对所有页面运行相同的`assertEquals`和`assertContains`函数。我们只是试图确认每个页面是否以 HTTP 200 加载，并且页面标题的正确值是否返回，也就是`h1`元素。

最后，我们需要在`DefaultControllerTest`类中添加以下内容来处理表单提交测试：

```php
public function testContactFormSubmit()
{
  // @var \Symfony\Bundle\FrameworkBundle\Client
  $client = static::createClient();
  /** @var \Symfony\Component\DomCrawler\Crawler */
  $crawler = $client->request('GET', '/contact');

  // Find a button labeled as "Reach Out!"
  $form = $crawler->selectButton('Reach Out!')->form();

  // Note this does not validate form, it merely tests against submission and response page
  $crawler = $client->submit($form);
  $this->assertEquals(200, $client->getResponse()->getStatusCode());
}
```

在这里，我们通过其**Reach Out!**提交按钮抓取表单元素。一旦获取表单，我们就在客户端上触发`submit`方法，将元素实例传递给它。值得注意的是，这里并没有测试实际的表单验证。即使如此，提交的表单应该会导致 HTTP 200 状态。

这些测试是有说服力的。如果我们愿意，我们可以编写更加健壮的测试，因为有许多元素可以进行测试。

# 总结

在本章中，我们构建了我们的第一个模块，或者在 Symfony 术语中称为 bundle。该模块本身并不是真正松散耦合的，因为它依赖于`app`目录中的一些内容，比如`app/Resources/views/base.html.twig`布局模板。当涉及核心模块时，我们可以这样做，因为它们只是我们为其余模块设置的基础。

在接下来的章节中，我们将构建一个目录模块。这将是我们网店应用程序的基础。

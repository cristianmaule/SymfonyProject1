# Symfony Guide

This document acts as a guide to Symfony for first time users

# Pre Reqs

- PHP 7.2
- MySQL 5.7
- [Composer](https://getcomposer.org/download/)
- [Symfony](https://symfony.com/download)

> On windows Latest version of WAMP seems to work well. As long as it's the only version of MySQL / PHP on the box!!! See link [WAMP](http://www.wampserver.com/en/)

If you are intending to use WebPack Encore then the following other dependencies are also needed

- [NodeJS](https://nodejs.org/en/download/) (Latest version)
- [Yarn Module](https://yarnpkg.com/lang/en/docs/install/#windows-stable) (Latest Version)  

# Creating the project structure

Following commands will create the base Symfony project

```bash
composer create-project symfony/website-skeleton <Project Name>
cd <Project Name>
```

# Core project dirs explanation

- `config/` Contains... configuration
- `src/` All your PHP code lives here.
- `templates/` All your Twig templates live here.
- `bin/` The famous bin/console file lives here (and other, less important executable files).
- `var/` This is where automatically-created files are stored, like cache files (var/cache/) and logs (var/log/).
- `vendor/` Third-party (i.e. "vendor") libraries live here! These are downloaded via the Composer package manager.
- `public/` This is the document root for your project: you put any publicly accessible files here.
- `assets/` This directory does not exist on initial project setup, but is created if using webpack. It's recommended things like CSS / JS / Images go here

# Add project to local Git Repository

Ermm we need source control!!!!

```bash
git init
git add .
git commit -m "Initial commit"
```

# Libraries

The following command can be used for adding Symfony libraries to your project

```bash
composer require <library name>
```

If it's an existing project you can load the libs using

```bash
composer install
```

# Common Libs

Below are some common libraries that are most likely needed for your project

```bash
composer require annotations
```

Annotations allow for adding annotations. This seems to be the preferred method in FOS! We prefer it because, rather than maintaining separate YAML files for setting up our endpoints, it allows us to tie said endpoints directly to code - so that we don't have to reference _another_ file when working out a path/endpoint's responsibilities.

```bash
composer require symfony/orm-pack
```

Installs doctrine ORM

```bash
composer require symfony/maker-bundle
```

Command line utility for making entities / controllers

```bash
composer require sensio/framework-extra-bundle
```

Allows annotations to be used for routes. Requires the (annotations) library.

```bash
composer require symfony/security-bundle
```

Allows for security controllers to be created, and for password encoding

> Note: See `Security` section of the guide later

```bash
composer require symfony/web-server-bundle
```

In built webserver to run your app!

```bash
composer require orm-fixtures
```

Allows creation of data fixture classes. Basically creating test data. This obviously needs the "orm-pack" installed! 

```bash
composer require symfony/profiler-pack
```
Not needed on production but is a very handy debugging tool. That sits as a tool bar on the bottom of each of your pages!

```bash 
composer require encore
yarn install
```
Installs `Webpack Encore` to allow for asset management. 
> **Important Note:** `yarn` is part of `NodeJS` not Symfony. So if using this you need to install `NodeJS` + `yarn` first as described in pre-reqs section.

# Database configuration

This section describes how to make use of doctrine to create / manage your DB

# Connection String

First step is to setup the connection string.

In the root of your project is a file called `.env`

In this file you need to modify the setting

```text
DATABASE_URL
```

As an example:

```text
# Configure your db driver and server_version in config/packages/doctrine.yaml
DATABASE_URL=mysql://root:the_root_password@127.0.0.1:3306/the_db_name
```

# Schema Creation

The following command is used to create the DB

```bash
php bin/console doctrine:database:create
```

# Entity Creation/Modification

The following command will guide you through Entity creation

```bash
php bin/console make:entity
```

Calling the above command and naming an existing Entity will start entity modification

> The following article will help with creating table relationships [Entity Relationships](https://symfony.com/doc/current/doctrine/associations.html)

Once created Entity classes are placed in following directory `<project_root>/src/Entity`

# Creating DB Tables From Entities

In Doctrine if you create / modify an Entity you will need to tell Doctrine to alter the actual schema based on your changes. This is done via a "Migration" system. Essentially Doctrine generates PHP files that run SQL code.
The SQL code is done in a version system so depending on how many changes you make you could end up with many migration file.

To generate the migration scripts you will need to run the following command

```bash
php bin/console make:migration
```

The above will only create the SQL script PHP files. It will NOT run them.
The PHP classes for each migration will appear in the directory `<project_root>/src/Migrations`

To apply all outstanding migrations the following command is used

```bash
php bin/console doctrine:migrations:migrate
```

# Creating Fixtures

Fixtures are classes used to generate test data for your database. Running the following command from the console will create a fixture shell

```bash
php bin/console make:fixtures
```

Once created the named fixture will appear in `<project_root>/src/DataFixtures`

When the fixture class is created a small modification is required to populate the data.

Inside the fixture will be a function called.

```php
    public function load(ObjectManager $manager)
    {
        // $product = new Product();
        // $manager->persist($product);

        $manager->flush();
    }
```

This needs changing to actually do something! For example:

```php
    $user = new User();
    $user->setEmail('cris.maule@bespokesoftware.com');
    $manager->persist($user);
    $manager->flush();
```

The above would create fill a user with mentioned email. Of course you could repeat the code creating multiple records.

> NOTE: Don't forget to add your entity as a USING statement at the top. (For example)

```php
    use App\Entity\User;
```

Once you have all your fixtures the following command will run them and populate the tables.

```bash
php bin/console doctrine:fixtures:load
```

> **Important Note:** This is an overwrite rather than an append!

# Generating the site!

So far the guide has helped create the project skeleton structure, and create/populate the DB with test data.
Now we need to actual get a site up and running!

We can run our site now using the following command

    php bin/console server:run

This will create a local webserver on a specific port. (although it's going to be a bit dull at the moment!)

# Security Getting Started Guide

Symfony has some in built functions to help you get up and running with a basic authentication system fairly quickly. If you have included the `security-bundle` as a library then you will be able to create a security controller and basic login form. The following article describes in detail this process.

> [Symfony Security](https://symfony.com/doc/current/security.html)

A very brief summary of the article is below:

The following command, is effectively the same as `make:entity`, but is a shorthand for creating a default `User` entity along with some other classes, and also updates the `security.yaml` file.

```bash
    php bin/console make:user
```

> **Important Note:** Once the above has been done you will still need to run DB migrations, with the commands below!

```bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

Now we have a base `User` entity class another quick action is to auto generate a login form

This is done with the following command.

```bash
php bin/console make:auth
```

This will generate a `controller` + `forms` needed for a simplistic login form.

The following article covers this in more detail.

> [Generating a login form](https://symfony.com/doc/current/security/form_login_setup.html)

# Password Encoding

Symfony provides the capability to encode passwords. The previous guide listed at the top of the `Security` section does cover this in more detail, but in short if you want to encode a password in code then you need the following code added to your class.

Firstly add the following USE statement

```php
    use Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface;
```

Inside your class add the following:

```php
    private $passwordEncoder;

    public function __construct(UserPasswordEncoderInterface $passwordEncoder)
    {
        $this->passwordEncoder = $passwordEncoder;
    }
```

Now when you want to encode a password all you need to do is

```php
    $this->passwordEncoder->encodePassword(
        $theEntity,
        'the_un_encoded_password_value_goes_here'
    )
```

> **Important Note:** `theEntitiy` must implement `UserInterface`

Lastly there is a handy command to just encode a password outside of code

```bash
php bin/console security:encode-password
```

# Controllers

To actually navigate around a Symfony site then a `Controller` is needed.
`Controllers`, as the name suggests, control the flow of the site.

If you have followed this guide so far then you will already have a `Controller` called `SecurityController`!
Controllers can be found in the directory `<project_root>/src/Controller`

The following command will help with creating a new `Controller`

```bash
php bin/console make:controller
```

Once you have your controller you can add public functions to it with routing annotations. (for example)

```php
    /**
     * @Route("/helloworld")
     */
    public function helloWorld()
    {
    }
```

Above adds route so URL with `/helloworld` on the end calls this function.

> **Important Note:** Annotations can only be used in `Controllers` if the `sensio/framework-extra-bundle` library is being used

Obviously as your site grows you will end up with many `Routes`. The following command is useful for listing all the existing `Routes` available

```bash
php bin/console debug:router
```

# Advanced Routing

You can pass Meta Data from URL to `Controller`. Effectively a REST style

```php
    /**
     * @Route("/product/edit/{id}")
     */
    public function edit($id)
    {
    }
```

following URL `/product/edit/4` will pass ID as 4 to edit function

Or if you are using: `sensio/framework-extra-bundle` library you can pass objects using same way, and DB will try and find that object with that id. (not tried it, but guessing you can change {id} with another field?)

```php
    /**
     * @Route("/product/{id}", name="product_show")
     */
    public function show(Product $product)
    {
    }
```

# Making a Controller do something!

In your controller function the following will render the `TWIG` file listed, and pass number var as an argument!
> `TWIG` will be covered in more detail later!

```php
    $number = random_int(0, 100);

    return $this->render('lucky/number.html.twig', [
        'number' => $number,
    ]);
```

Or it can return JSON

```php
    return $this->json(['username' => 'jane.doe']);
```

You can also get hold of the `Request` object inside a `Controller`.

```php
    use Symfony\Component\HttpFoundation\Request;

    public function index(Request $request, $firstName, $lastName)
    {
        $page = $request->query->get('page', 1);
    }
```

# Redirect to another route from within code

All the following examples need the following USE statement

```php
    use Symfony\Component\HttpFoundation\RedirectResponse;
```

A simple example

```php
    return $this->redirectToRoute('homepage');
```

Or with args:

```php
    return $this->redirectToRoute('app_lucky_number', ['max' => 10]);
```

Or with all original args

```php
    return $this->redirectToRoute('blog_show', $request->query->all());
```

Or go outside of the site!

```php
    return $this->redirect('http://symfony.com/doc');
```

# Templating / Assets

So we have some routes but not an actual proper site. 
Now we need to create the pages needed by the site.

# Handling Assets with WebPack Encore

`Webpack Encore` is a symfony addon that makes handling of assets `CSS/JS/Images` easier.
You can if you want just include all your assets inside the `/public` folder. However this is not best practice. It is instead suggested that you put your files inside `/assets` and use `Webpack Encore` to manage assets.

> **Important Note:** `Webpack Encore` requires `NodeJS` / `Yarn` to be able to operate

The following guide [Installing/Configuring Webpack Encore](https://symfony.com/doc/current/frontend.html) details how to implement/use, but the basics are as follows.

If it's installed correctly you should already have a `/assets` folder with some `CSS / JS` in there already.
Likewise it's configuration file will now be created called `/webpack.config.js`

The following command will compile, and send assets to the public folder once.

```bash
yarn encore dev
```

If you want to keep watching for changes you can run

```bash
yarn encore dev --watch
```

> **Important Note:** The above will tie up a terminal session!

# Using WebPack Encore in your templates.

Now we've got `Webpack Encore` installed it's time to use it in your code.

First edit the base `twig` template found here `/templates/base.html.twig`.
By default this will have an empty set of `stylesheets` / `javascripts` blocks.

All we need to do is modify the `stylesheets` block to become

```twig
{% block stylesheets %}
    {{ encore_entry_link_tags('app') }}
{% endblock %}
```

The `javascripts` block needs to be modified as per the below

```twig
{% block javascripts %}
    {{ encore_entry_script_tags('app') }}
{% endblock %}
```

# Copying images with WebPack Encore 
By default `Webpack Encore` only copies certain files. For example just dropping files in won't do anything. For examples images.

If you do wish to do items such as images then you will need to modify the `/wepack.config.js` file to do so.

As a very basic example the following lines added to the config as part of the chaining in the `Encore` function will copy all images from `/assets/images`

```js
    .copyFiles({
         from: './assets/images'
    })
```
The following article covers the above in more details. [Copy Files With Webpack Encore](https://symfony.com/doc/current/frontend/encore/copy-files.html)

# Converting WebPack from CSS to SCSS

By default the instalation of `Webpack Encore` is built around `JS / CSS`
To convert to SCSS rename the `/assets/css/app.css` file to `/assets/app.scss`

We also need to update `/assets/js/app.js` to point at the new file.

Change the `require` line in the file `app.js` file from `.css` to `.scss` as per the below

```js
require('../css/app.scss');
```

Next we need to modifiy the `/wepack.config.js` file.

firstly uncomment the line

```js
.enableSassLoader()
```

Now run the following command to install SASS

```bash
yarn add sass-loader@^7.0.1 node-sass --dev
```

Lastly re-run the command below to re-deploy

```bash
yarn encore dev
``` 

# TWIG Templates

# Installing Bootstrap

If you want to install `Boostrap` then you can do so with the following commands.

> **Important Note:** This is assuming you are using the `Webpack Encore` and have converted to `scss` first!

First install `Boostrap`

```bash
yarn add bootstrap --dev
```

Next we need to modify the `/assets/app.scss` file to include `Bootstrap`.

Simply add the following line into the `.scss` file

```scss
@import "~bootstrap/scss/bootstrap";
```

You can even customize `Bootstrap` vars from the `.scss` file also. see example below

```scss
// customize some Bootstrap variables
$primary: darken(#428bca, 20%);

@import "~bootstrap/scss/bootstrap";
```

Or you can override the bootstrap files in the .scss file after the bootstrap import as per below example

```scss
// the ~ allows you to reference things in node_modules
@import "~bootstrap/scss/bootstrap";

body {
    background-color: orange;
}

.btn-primary {
    color: #fff;
    background-color: #3333ff;
    border-color: #245682;
}
```

That's it now you can use `Boostrap` as you normally would. For example

```twig
<button class="btn btn-lg btn-primary mt-3" type="submit">
    Sign in
</button>
```

# Bootstrap Javascript
If you want to use `Boostrap Javascript` for exmaple `popper` then you need to modify your `.js` file.

First install the `js` module needed. The following is just for `popper`

```bash
yarn add jquery popper.js
```

Now edit your `.js` file. This example was done with `/assets/js/app.js`

```js
const $ = require('jquery');

require('bootstrap');

$(document).ready(function() {
    $('[data-toggle="popover"]').popover();
});
```

Now in your `twig` template you can use it as you normally would as per the below

```twig
<button type="button" class="btn btn-lg btn-danger" data-toggle="popover" title="Popover title" data-content="And here's some amazing content. It's very engaging. Right?">Click to toggle popover</button>
```

Other example packages!
```bash
yarn add scrollspy
```

# Installing FontAwesome
If you wish to use `FontAwesome` you can do so with the following commands

> **Important Note:** This is assuming you are using the `Webpack Encore` & `Boostrap` bundle and have converted to `scss` first!

First install `Font Awesome`
```bash
yarn add --dev @fortawesome/fontawesome-free
```

> **Important Note:** The "FORT" is in the above is NOT a typo. Do NOT rename to "FONT" !!!!!!

Require font-awesome into your config file `assets/js/app.js`:

```js
require('@fortawesome/fontawesome-free/css/all.min.css');
require('@fortawesome/fontawesome-free/js/all.js');
```

Compile again using following command to copy font's to the public folder. 

```bash
yarn encore dev 
```

To use a font in your template you can do so as you normally would

```htm
<i class="fa fa-users" aria-hidden="true"></i>
```

See following articles for full details:

> [Font Awesome Installation Using Package Manager](https://fontawesome.com/how-to-use/on-the-web/setup/using-package-managers)

> [Installing FontAwesome 5 in Symfony 4 with Webpack Encore (Stack Overflow)](https://stackoverflow.com/questions/51571170/how-to-add-fontawesome-5-to-symfony-4-using-webpack-encore)

# TWIG & Assets
# Handling Assets outside of the public site
# TWIG & overrides
# TWIG & Clientside JS
# TWIG & CSS
# Forms

# Transformers

# Command (CRON)

# Autowiring

# Repositories

# Appendix Handy Guides!

https://symfony.com/doc/current/frontend/encore/bootstrap.html

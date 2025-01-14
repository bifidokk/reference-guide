# Before we start tutorial

## Setup for tutorial

Depending on your preferences, you may choose for the tutorial [`Symfony`](https://symfony.com)  [`Laravel`](https://laravel.com) or `Lite (No extra framework)`.\
If configuration will differ, there will be three tabs available, each with code specific to the chosen solution.

1. Use [git](https://git-scm.com) to download  **a starting point** to follow the tutorial.

{% tabs %}
{% tab title="Symfony" %}
```
git clone git@github.com:ecotoneframework/symfony-tutorial.git
# Go to symfony-tutorial catalog
```
{% endtab %}

{% tab title="Laravel" %}
```bash
git clone git@github.com:ecotoneframework/laravel-tutorial.git
# Go to laravel-tutorial catalog

# Normally you will use "php artisan" for running console commands
# To reduce number of difference during the tutorial
# "artisan" is changed to "bin/console"
```
{% endtab %}

{% tab title="Lite" %}
```
git clone git@github.com:ecotoneframework/lite-tutorial.git
# Go to lite-tutorial catalog
```
{% endtab %}
{% endtabs %}

2\. Run command line application. There are two options, run on your _Local Environment_ or using _Docker_

{% tabs %}
{% tab title="Local Environment" %}
```php
/** You need to have atleast PHP 8.0 and Composer installed */
1. Run "composer install" 
2. Run starting command "bin/console ecotone:quickstart"
3. You should see:
"Running example...
Hello World
Good job, scenario ran with success!"
```
{% endtab %}

{% tab title="Docker" %}
```php
/** Ecotone Quickstart ships with docker-compose with preinstalled PHP 8.0 */
1. Run "docker-compose up -d"
2. Enter container "docker exec -it ecotone-quickstart /bin/bash"
3. Run starting command "composer instal"
4. Run starting command "bin/console ecotone:quickstart"
5. You should see:
"Running example...
Hello World
Good job, scenario ran with success!"
```
{% endtab %}
{% endtabs %}

{% hint style="success" %}
Great, we are prepared for Lesson 1!
{% endhint %}

{% content-ref url="php-messaging-architecture.md" %}
[php-messaging-architecture.md](php-messaging-architecture.md)
{% endcontent-ref %}

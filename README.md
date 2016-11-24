Bridge
===============
2016-11-24



Pattern for handling services in your application.





If you know what a service container is in services oriented architectures, then Bridge is nothing more than a lightweight service container.

If you don't know what a service container is, then read on...





Why?
------------

If you want your application to be modular, then you have to create some kind of modules.

Your application will use different methods to which modules can register.


For instance, imagine you want to display some html links in the left menu;
your application has a displayLeftMenuLinks method, and you would like your modules to be able to hook inside this method.

Another example is if you have a class for handling the permissions in your application (like [Privilege](https://github.com/lingtalfi/Privilege) for instance),
and you want your application modules to be able to define their own privileges.


So in other words, your application will call highly specialized services, and you would like your modules to be able to subscribe to those services.


You need to think about how you could make this happen.

Guess what, I've just thought about it so you can use my solution if you like: it's called Bridge, and it's very simple.




Conception
---------

If you are interested in why I chose this design over another one, this section is for you.

In php there are two kinds of method calls: static and dynamic (non static).

With static methods, life is easy, you can just call the static method from nowhere, nice.

The problem comes with dynamic methods, because you need a class instance.

Since a service could be called multiple times, the last thing I wanted was to have to re-instantiate a class on every service call.

The solution is quite simple: save the instance in an array and re-use the instance if it's in the array.


### Why hardcoded?

What makes Bridge unique is that you customize the code directly.

This is now the way I like it, because you know exactly what's going on, you operate at the php source code level.

So, there is a direct and obvious link between your code in the Bridge you and the application, which makes the application intuitive to use for a developer,
and simple because there is no overhead.



How?
------------

Now let's learn how to use the Bridge class.

The Bridge class itself is so shim that I can just display it here:



```php
<?php


class Bridge
{


    private static $instances = [];



    //--------------------------------------------
    // APPLICATION SERVICES
    //--------------------------------------------
    public static function displayLeftLinks()
    {
        MyOtherClass::displayLinks();
        self::getInstance('Bob')->displayLinks();
    }








    //--------------------------------------------
    // INSTANCES PREPARATION
    //--------------------------------------------
    private static function getBob()
    {
        return new Bob();
    }



    //--------------------------------------------
    // PRIVATE
    //--------------------------------------------
    private static function getInstance($name)
    {
        if (!array_key_exists($name, self::$instances)) {
            self::$instances[$name] = call_user_func('Bridge::get' . $name);
        }
        return self::$instances[$name];

    }



}
```



As you can see there are three sections:

- application services
- instances preparation
- private


There is also an instances array at the top.


In the "Applications services" section, you will put all the hookable methods used by your application.

You will probably get rid of the displayLeftLinks example method and put your owns here.

The application services should be only static php methods, so that the call to a service is easy from your application.

Inside an application service, that's where you hook your modules.

You can either use the static way (if your module provide a static method), or the dynamic way if your module only uses a dynamic method.

If you look inside the example displayLeftLinks method, you will see that both cases are implemented.

The first line:

```php
MyOtherClass::displayLinks();
```

is an example of how you would hook one of your module to the displayLeftLinks service.

It's straight forward and need no further explanations.

The second line:

```php
self::getInstance('Bob')->displayLinks();
```

show how you use a dynamic method.

In order to call a static method, you need some preparation first:

- create a private static method which name starts with "get" in the "instances preparation" section (the second section).
		You can use the getBob example as a model that you can duplicate.
		Note: if your instance needs instantiation parameters, you will put them here directly.

Once this "instance preparation" method is created, you can now use the getInstance method inside the displayLeftLinks service.
Just omit the get prefix.

So, the line:

```php
self::getInstance('Bob')->displayLinks();
```
basically calls the static 'getBob' "instance preparation" method that you've created in advance, and call the displayLinks dynamic method of the Bob instance.

It also stores the Bob instance in memory, so that the next time the displayLeftLinks service is called, it will not re-instantiate the Bob class.


I'm sure my explanations make it more difficult to understand that it really is (I'm sorry for that), but I'm also confident that you will read the source code
and understand the pattern intuitively.





























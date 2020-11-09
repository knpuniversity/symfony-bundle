# Bootstrapping the Bundle & Autoloading

Heeeeey Symfony peeps! I'm excited! Because we're going to dive *deep* in to a
*super* interesting topic: how to create your *own* bundles. This is useful if you
need to share code between your own projects. Or if you want to share your great
new open source library with the *whole* world. Actually, forget that! This tutorial
is going to be *awesome* even if you *don't* need to do either of those. Why? Because
we *use* third-party bundles *every* day. And by learning how to create one, we're
going to become experts in how they work and *really* get a look at Symfony under
the hood.

As always, you can earn free high-fives by downloading the source code from this
page and coding along with me. After unzipping the file, you'll find a `start/` directory
with the same code that you see here. Follow the `README.md` file for steps on how
to get your project setup.

The last step will be to open a terminal, move into the project, sip your coffee,
and run:

```terminal
php bin/console server:run
```

to start the built-in PHP web server.

## Introducing KnpUIpsum

Head to your browser and go to `http://localhost:8000`. Say hello to The Space Bar!
This is a Symfony *application* - the one we're building in our beginner Symfony
series. Click into one of the articles to see a *bunch* of *delightful*, fake text
that we're using to make this page look real. Each time you refresh, you get new
random, happy content.

To find out where this is coming from, in the project, open `src/Service/KnpUIpsum.php`.
Yes! This is our *new* creation: it returns "lorem ipsum" dummy text, but with
a *little* KnpUniversity flare: the classic latin is replaced with rainbows, unicorns,
sunshine and more of our favorite things.

[[[ code('d8e0cedd2e') ]]]

And, you know what? I think we *all* deserve more cupcakes, kittens & baguettes
in our life. So I want to share this functionality with the world, by creating the
KnpULoremIpsumBundle! Yep, we're going to extract this class into its own bundle,
handle configuration, add tests, and do a bunch of other cool stuff.

Right now, we're using this code inside of `ArticleController`: it's being passed
to the constructor. Below, we use that to generate the content.

[[[ code('e2acaafad8') ]]]

## Isolating into a new Bundle Directory

Ok, the *first* step to creating a new bundle is to move this code into its own
location. *Eventually*, all the code for the bundle will live in its own *completely*
separate directory & repository. But, sometimes, when you *first* start building,
it's a bit easier to *keep* the code in your project: it let's you hack on things
really quickly & test them in your app.

So let's keep the code here for now, but isolate it from the app's code. To do that,
create a new `lib/` directory. And then, another called `LoremIpsumBundle`: this
will be the temporary home for our shiny bundle. Inside, there are a few valid
ways to organize things, but I like to create a `src/` directory.

```terminal-silent
mkdir lib
mkdir lib/LoremIpsumBundle
mkdir lib/LoremIpsumBundle/src
```

***TIP
You can also just type one command instead of three:

```terminal-silent
mkdir -p lib/LoremIpsumBundle/src
```
***

Perfect! Now, move the `KnpUIpsum` class into that directory. And yea, you could
put this into a `src/Service` directory, or anywhere else you want.

## New Vendor Namespace

Oh, but this namespace will *not* work anymore. We need a namespace that's *custom*
to our bundle. It could be anything, but usually it has a vendor part - like
`KnpU` and then the name of the library or bundle - `LoremIpsumBundle`.

[[[ code('3bf3f18617') ]]]

And, that's it! If we had decided to put `KnpUIpsum` into a sub-directory, like `Service`,
then we would of course also add `Service` to the end of the namespace like normal.

Next, back in `ArticleController`, go up to the top, remove the use statement, and
re-type it to get the new one.

[[[ code('0f4c53c768') ]]]

## Handling Autoloading

So... will it work! Yea... probably not - but let's try it! Nope! But I *do* love
error messages:

> Cannot autowire ArticleController argument $knpUIpsum... because the KnpUIpsum
> class was not found.

Of course! After creating the new `lib/` directory, we need to tell Composer's
autoloader to look for the new classes there. Open `composer.json`, find the
`autoload` section, and add a new entry: the `KnpU\\LoremIpsumBundle\\` namespace
will live in `lib/LoremIpsumBundle/src/`.

[[[ code('f1009bd636') ]]]

Then, open a new terminal tab. To make the autoload changes take effect, run:

```terminal
composer dump-autoload
```

## Registering the Service

Will it work *now*? Try it! Bah, not yet: but we're closer. The error changed:
instead of "class not found", now it says that no `KnpUIpsum` service exists.
To solve this, open `config/services.yaml`.

Thanks to the auto-registration code in here, we don't normally need to register
our classes as services: that's automatic. But, it's only automatic for classes
that live in `src/`. Yep, as *soon* as we moved the class from `src/` to `lib/`,
that service disappeared.

And that's ok! When you create a re-usable bundle, you actually *don't* want to
rely on auto-registration or autowiring. Instead, as a best-practice, you should
configure everything *explicitly* to avoid any surprises.

To do that, at the bottom of this file, add `KnpU\LoremIpsumBundle\KnpUIpsum: ~`.

[[[ code('1c16809db1') ]]]

This adds a new service for that class. And because we don't need to pass any
options or arguments, we can just set this to `~`. The class *does* have constructor
arguments, but they have default values.

Ok, try it again! Yes! It *finally* works! We've successfully isolated our code
into its own directory and we are ready to hack! Next, let's make this a bundle
with a bundle class and start digging into how bundles can automatically register
services.

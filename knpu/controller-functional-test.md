# Controller Functional Test

We just added a route and controller, and since this bundle is going to be used
by, probably, *billions* of people, I want to make sure they work! How? By
writing a good old-fashioned functional test that surfs to the new URL and checks
the result.

In the `tests/` directory, create a new `Controller` directory and a new PHP class
inside called `IpsumApiControllerTest`. As always, make this extend `TestCase` from
PHPUnit, and add a `public function testIndex()`.

[[[ code('590a77c155') ]]]

## How to Boot a Fake App?

The setup for a functional test is pretty similar to an integration test: create
a custom test kernel, but this time, import `routes.xml` inside. Then, we can use
Symfony's BrowserKit to make requests into that kernel and check that we get a 200
status code back.

Start by stealing the testing kernel from the `FunctionalTest` class. Paste this
at the bottom, and, just to avoid confusion, give it a different name:
`KnpULoremIpsumControllerKernel`. Re-type the `l` and hit tab to add the `use`
statement for the `Kernel` class.

[[[ code('8ab03d7519') ]]]

Then, we can simplify: we don't need any special configuration: just call the parent
constructor. Re-type the bundle name and hit tab to get the use statement, and
do this on the other two highlighted classes below. Empty the `load()` callback
for now.

Yep, we're just booting a kernel with one bundle... super boring.

## Do we Need FrameworkBundle Now?

And here's where things get confusing. In `composer.json`, as you know, we do
*not* have a dependency on `symfony/framework-bundle`. But now... we have a route
and controller... and... well... the *entire* routing and controller system comes
from FrameworkBundle! In other words, while not *impossible*, it's incredibly unlikely
that someone will want to import our route, but *not* use FrameworkBundle.

This means that we *now* depend on FrameworkBundle. Well actually, that's not *entirely*
true. Our new route & controller are optional features. So, in a perfect world,
FrameworkBundle should *still* be an *optional* dependency. In other words, we are
*not* going to add it to the `require` key. In reality, if you did, no big deal -
but we're doing things the harder, more interesting way.

This leaves us with a big ugly problem! In order to *test* that the route and
controller work, we need the route & controller system! We need
FrameworkBundle! This is yet *another* case when we need a dependency, but we *only*
need the dependency when we're developing the bundle or running tests. Find your
terminal and run:

```terminal
composer require symfony/framework-bundle --dev
```

Let this download. Excellent!

## Importing Routes from the Kernel

Back in the test, thanks to FrameworkBundle, we can use a *really* cool trait to
make life simpler. Full disclosure, I helped created the trait - so of course *I*
think it's cool. But really, it makes life easier: `use MicroKernelTrait`. Remove
`registerContainerConfiguration()` and, instead go back again to the
Code -> Generate menu - or Command + N on a Mac - and implement the two missing
methods: `configureContainer()`, and `configureRoutes()`.

[[[ code('158d5285f9') ]]]

Cool! So... let's import our route! `$routes->import()`, then the path to that
file: `__DIR__.'/../../src/Resources/config/routes.xml'`.

[[[ code('667e979508') ]]]

## Setting up the Test Client

Nice! And... that's really all the kernel needs. Back up in `testIndex()`, create
the new kernel: `new KnpULoremIpsumControllerKernel()`.

[[[ code('bc64fbe792') ]]]

Now, you can almost pretend like this a normal functional test in a normal Symfony
app. Create a test client: `$client = new Client()`  - the one from FrameworkBundle -
and pass it the `$kernel`.

***TIP
In Symfony 4.3 and higher, use `KernelBrowser` instead of `Client`: the class was renamed.
***

Use this to make requests into the app with `$client->request()`. You will *not*
get auto-completion for this method - we'll find out why soon. Make a `GET` request,
and for the URL... actually, down in `configureRoutes()`, ah, I forgot to add a prefix!
Add `/api` as the second argument. Make the request to `/api/`.

[[[ code('8f094f9fa4') ]]]

[[[ code('1fdd5cb426') ]]]

Cool! Let's dump the response to see what it looks like:
`var_dump($client->getResponse()->getContent())`. Then add an assert that 200
matches `$client->getResponse()->getStatusCode()`.

[[[ code('9cc225f5d4') ]]]

Alright! Let's give this a try! Find your terminal, and run those tests!

```terminal-silent
./vendor/bin/simple-phpunit
```

Woh! They are *not* happy:

> Fatal error class `BrowserKit\Client` does not exist.

Hmm. This comes from the `http-kernel\Client` class. Here's what's happening:
we use the `Client` class from FrameworkBundle, *that* extends `Client` from
`http-kernel`, and *that* tries to use a class from a component called `browser-kit`,
which is an *optional* dependency of `http-kernel`. Geez.

Basically, we're trying to use a class from a library that we don't have installed.
You know the drill, find your terminal and run:

```terminal
composer require symfony/browser-kit --dev
```

When that finishes, try the test again!

```terminal-silent
./vendor/bin/simple-phpunit
```

Oof. It just looks *awful*:

> LogicException: Container extension "framework" is not registered.

This comes from `ContainerBuilder`, which is called from somewhere inside `MicroKernelTrait`.
This is a bit tougher to track down. When we use `MicroKernelTrait`, behind the
scenes, it adds some `framework` configuration to the container in order to configure
the router. But... our kernel does *not* enable FrameworkBundle!

No problem: add `new FrameworkBundle` to our bundles array.

[[[ code('e48c023862') ]]]

Then, go back and try the tests again: hold your breath:

```terminal-silent
./vendor/bin/simple-phpunit
```

No! Hmm:

> The service url_signer has a dependency on a non-existent parameter "kernel.secret".

This is a fancy way of saying that, for *some* reason, there is a missing parameter.
It turns out that FrameworkBundle has *one* required piece of configuration. In
your application, open `config/packages/framework.yaml`. Yep, right on top: the
`secret` key.

This is used in various places for security, and, since it needs to be unique and
secret, Symfony can't give you a default value. For our testing kernel, it's meaningless,
but it needs to exist. In `configureContainer()`, add `$c->loadFromExtension()`
passing it `framework` and an array with `secret` set to anything. The `FrameworkExtension`
uses this value to set that missing parameter.

[[[ code('ba74b96ed6') ]]]

Do those tests... one, last time:

```terminal-silent
./vendor/bin/simple-phpunit
```

Phew! They *pass*! The response status code is 200 and you can even see the JSON.
Go back to the test and take out the `var_dump()`.

Next, let's get away from tests and talk about events: the *best* way to allow
users to hook into your controller logic.

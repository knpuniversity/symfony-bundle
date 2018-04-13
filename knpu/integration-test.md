# Service Integration Test

Thanks to the unit test, we can confidently say that the `KnpUIpsum` class works
correctly. But... that's only like 10% of our bundle's code! *Most* of the bundle
is related to service configuration. So what guarantees that the bundle, extension
class, Configuration class and `services.xml` files are all correct? Nothing! Yay!

And it's not that we need to test *everything*, but it would be great to *at least*
have a "smoke" test that made sure that the bundle correctly sets up a
`knpu_lorem_ipsum.knpu_ipsum` service.

## Bootstrapping the Integration Test

We're going to do that with a functional test! Or, depending on how you name things,
this is really more of an integration test. Details. Anyways, in the `tests/` directory,
create a new class called `FunctionalTest`.

Make this extend the normal `TestCase` from PHPUnit, and add a
`public function testServiceWiring()`.

[[[ code('a1f4d1ed52') ]]]

And here is where things get interesting. We basically want to initialize our bundle
into a real app, and check that the container has that service. But... we do *not*
have a Symfony app lying around! So... let's make the *smallest* possible Symfony
app ever.

To do this, we just need a Kernel class. And instead of creating a new *file* with
a new class, we can hide the class right inside *this* file, because it's
only needed here.

Add `class KnpULoremIpsumTestingKernel extends Kernel` from... wait... why is this
not auto-completing the `Kernel` class? There *should* be one in Symfony's HttpKernel
component! What's going on?

## Dependencies: symfony/framework-bunde?

Remember! In our `composer.json`, other than the PHP version, the `require` key
is empty! We're *literally* saying that someone is allowed to use this bundle even
if they use *zero* parts of Symfony. That's not OK. We need to be explicit about
what dependencies are *actually* required to use this bundle.

But... what dependencies are required, exactly? Honestly... most bundles simply
require `symfony/framework-bundle`. FrameworkBundle provides all of the core services,
like the router, session, etc. It *also* requires the `http-kernel` component,
`event-dispatcher` and probably anything else that your bundle relies on.

Requiring FrameworkBundle is *not* a horrible thing. But, it's *technically*
possible to use the Symfony framework *without* the FrameworkBundle, and some
people *do* do this.

So we're going to take the *tougher*, more interesting road and *not* simply
require that bundle. Instead, let's look at the actual components our code uses.
For example, open the bundle class. Obviously, we depend on the `http-kernel`
component. And in the extension class, we're using `config` and `dependency-injection`.
In `Configuration`, nothing new: just `config`.

Ok! Our bundle needs the `config`, `dependency-injection` and `http-kernel` components.
And by the way, this is *exactly* why we're writing the integration test! Our bundle
is not setup correctly right now... but it wasn't very obvious.

## Adding our Dependencies

In `composer.json`, add these: `symfony/config` at version `^4.0`. Copy this
and paste it two more times. Require `symfony/dependency-injection` and
`symfony/http-kernel`.

[[[ code('c0f567e928') ]]]

Now, find your terminal, and run:

```terminal
composer update
```

Perfect! Once that finishes, we can go back to our functional test. Re-type
the "l" on `Kernel` and... yes! *There* is the Kernel class from `http-kernel`.

This requires us to implement two methods. Go to the Code -> Generate menu - or
Command + N on a Mac - click "Implement Methods" and choose the two.

[[[ code('94d7b97d78') ]]]

Inside `registerBundles`, return an array and *only* enable *our* bundle:
`new KnpULoremIpsumBundle()`. Since we're not dependent on any other bundles - like
`FrameworkBundle` - we should, in theory, be able to boot an app with only this.
Kinda cool!

[[[ code('8e2b225603') ]]]

And... that's it! Our app is ready. Back in `testServiceWiring`, add
`$kernel = new KnpULoremIpsumTestingKernel()` and pass this `test` for the environment,
thought that doesn't matter, and `true` for debug. Next, *boot* the kernel, and
say `$container = $kernel->getContainer()`.

[[[ code('70530682bd') ]]]

This is *great*! We just booted a *real* Symfony app. And now, we can makes sure
our service exists. Add `$ipsum = $container->get()`, copy the id of our service,
and paste it here. We can do this because the service is public.

Let's add some very basic checks, like `$this->assertInstanceOf()` that
`KnpUIpsum::class` is the type of `$ipsum`. And also, `$this->assertInternalType()`
that a string is what we get back when we call `$ipsum->getParagraphs()`.

[[[ code('9a59c7dba3') ]]]

The unit test *truly* tests this class - so we really only need a sanity check.
I think it's time to try this! Find your terminal, and run:

```terminal
./vendor/bin/simple-phpunit
```

Yes! We're now *sure* that our service is wired correctly! So, this functional test
didn't *fail* like I promised in the last chapter. But the point is this: before
we added our dependencies, our bundle was *not* actually setup correctly.

And, woh! In the `tests/` directory, we suddenly have a `cache/` folder! That
comes from our kernel: it caches files just like a normal app. To make sure
this doesn't get committed, open `.gitignore` and ignore `/tests/cache`.

[[[ code('78feaee4fa') ]]]

Next, let's get a little more complex by testing that some of our configuration
options work.

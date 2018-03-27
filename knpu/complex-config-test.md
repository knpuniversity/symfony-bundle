# Complex Config Test

There is *one* important part of the bundle that is *not* tested yet: our
configuration. I mean, if the user sets the `min_sunshine` option, there's no
test that this is correctly passed to the service.

And yea, again, you do *not* need to have a test for *everything* - use your best
judgment. For configuration like this, there are *three* different ways to test
it. First, you can test the `Configuration` class itself. That's a nice idea if
you have some really complex rules. Second, you can test the extension class directly.
In this case, you would pass different config arrays to the `load()` method and
assert that the arguments on the different `Definition` objects of different services
are set correctly. It's a really low level test, but it works.

And *third*, you can test your configuration with an integration test like we created,
where you boot a real application with some config, and check the behavior of the
final services.

Of you *do* want to test the configuration class or the extension class, like always,
a great way to do this is by looking at the core code. Press Shift+Shift to open
`FrameworkExtensionTest`. If you did some digging, you'd find out that this test
parses YAML files full of `framework` configuration, parses them, then checks to
make sure the `Definition` objects are correct.

Try Shift + Shift again to open `ConfigurationTest`. There are a bunch of these,
but the one from `FrameworkBundle` is a pretty good example.

## Dummy Test Word Provider

We're going to use the third option: boot a *real* app with some config, and test
the final services. Specifically, I want to test that the custom `word_provider`
config functionality.

Let's think about this: to create a custom word provider, you need the class,
like `CustomWordProvider`, you need to register it as a service - which is automatic
in our app - and *then* you need to pass the service id to the `word_provider`
option. We're going to do *all* of that, right here at the bottom of this test
class. It's a little nuts, which is why we're talking about it!

Create a new class called `StubWordList` and make it implement `WordProviderInterface`.
This will be our fake word provider. Go to the Code -> Generate menu, or Command + N
on a Mac, and implement the `getWordList()` method. Just return an array with two
words: `stub` and `stub2`.

Next, copy the `testServiceWiring()` method, paste it, and rename it to
`testServiceWiringWithConfiguration()`. Remove the last two asserts: we're going
to work more on this in a minute.

Here's the tricky part: we're using the same kernel in two different test... but
we want them to *behave* differently. In the second test, I need to pass some extra
configuration. This will look a bit technical, but just follow me through this.

First, inside the kernel, go back to the Code -> Generate menu, or Command + N on
a Mac, and override the constructor.To simplify, instead of passing the environment
and debug flag, just hard-code those when we call the parent constructor.

Thanks to that, we can remove those arguments in our two test functions above.
But *now*, add an optional array argument called `$knpUIpsumConfig`. This wil be
the configuration we want to pass under the `knpu_lorem_ipsum` key.

At the top of the kernel, create a new private variable called `$knpUIpsumConfig`,
and then assign that in the constructor to the argument.

Next, find `registerContainerConfiguration()` method. In a normal Symfony app,
*this* is the method that's responsible for parsing all the YAML files in the
`config/packages` directory and the `services.yaml` file.

Instead of parsing YAML files, we can instead put all that logic into PHP with
`$loader->load()` passing it a callback function with a `ContainerBuilder` argument.
Inside of *here*, we can start registering services and passing bundle extension
configuration.

First, in all cases, let's register our `StubWordList` as a service:
`$container->register()`, pass it any id - like `stub_word_list` - and pass the
class: `StubWordList::class`. It doesn't need any arguments.

Next, we want to pass any custom `knpu_lorem_ipsum` bundle extension configuration.
Do this with `$container->loadFromExtension()` with `knpu_lorem_ipsum` and, for
the second argument, the array of config you want: `$this->knpUIpsumConfig`.

Basically, each test case can *now* pass in whatever custom config they want. The
first won't pass any, but the second will pass the `word_provider` key set to
the service id: `stub_word_list`.

The *downside* of an integration test is that we can't assert *exactly* that the
`StubWordList` was passed into `KnpUIpsum`. We can only test the *behavior* of
the services. But since that stub word list only uses two different words, we
can reasonably test this with `$this->assertContains('stub', $ipsum->getWords(2))`.

Ready to try this? Find your terminal and... run those tests!

```terminal-silent
./vendor/bin/simple-phpunit
```

Ah man! Our new test *fails*! It looks like it's *not* using our custom word provider.
Weird!

It's probably weirder than you think. Re-run *just* that test by passing
`--filter testServiceWiringWithConfiguration`:

```terminal-silent
./vendor/bin/simple-phpunit --filter testServiceWiringWithConfiguration
```

It still fails. But now, clear the cache directory:

```terminal
rm -rf tests/cache
```

And try the test again:

```terminal-silent
./vendor/bin/simple-phpunit --filter testServiceWiringWithConfiguration
```

Holy Houdini Batman! It *passed*! In fact, try *all* the tests:

```terminal-silent
./vendor/bin/simple-phpunit
```

That *all* pass! Black magic! What the heck just happened?

Ok, when you boot a kernel, it creates a `tests/cache` directory that includes the
cached container. The *problem* is that it's using the same cache directory for
*both* of these kernels. Once the cache directory is populated the first time, *all*
future tests re-use that same container, instead of building their own.

It's a subtle problem, but has an easy fix: we need to make the `Kernel` use a
different cache directory each time it's instantiated. There are infinite number
of ways to do this, but here's an easy one. Go back to the Code -> Generate menu,
or Command + N on a Mac, an override a method called `getCacheDirectory()`. Return
`__DIR__.'/cache/` then `spl_object_hash($this)`. So, we will *still* use that
cache directory, but each time you create a new Kernel, it will use a different
subdirectory.

Clear out the cache directory one last time. Then, run the tests!

```terminal-silent
./vendor/bin/simple-phpunit
```

They pass! Run them again:

```terminal-silent
./vendor/bin/simple-phpunit
```

You'll now see *four* unique sub-directories inside `cache/`. I won't do it, but
to make things even better, you could even clear the `cache/` directory between
tests with a `teardown()` method in the test class.

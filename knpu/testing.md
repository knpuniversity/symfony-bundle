# Testing the Bundle

Hey! Someone already made some tests for our bundle! *So* nice! Right now, they live
in the *app*, but moving them *into* the bundle is our next job! But first... let's
make sure they're still working.

[[[ code('36083c6e13') ]]]

Find the terminal tab for the application and run:

```terminal
./vendor/bin/simple-phpunit
```

The first time you run this, it'll download PHPUnit behind the scenes. Then...
it does *not* pass!

***TIP
The `assertInternalType()` method has been removed, you can use `assertIsString()` instead:
```php
$this->assertIsString($words);
```
If you want to know more about this: https://github.com/sebastianbergmann/phpunit/issues/3369
***

> Class `App\Service\KnpUIpsum` not found

Of course! When we moved this class into the new namespace, we did *not* update
the test! No problem - just re-type `KnpUIpsum` and hit tab to auto-complete
and get the new `use` statement.

[[[ code('7d5fdc63b7') ]]]

Perfect! But... I can already see another problem! When we added the first constructor
argument to `KnpUIpsum`, we *also* didn't update the test. I could use mocking here,
but it's just as easy to say `new KnpUWordProvider`. Repeat that in the two other
places.

[[[ code('bce2f18675') ]]]

Ok, try those tests again!

```terminal-silent
./vendor/bin/simple-phpunit
```

Got it!

## Adding Tests to your Bundle & autoload-dev

Time to move this into our bundle. We already have a `src/` directory. Now create
a new directory next to that called `tests/`. Copy the `KnpUIpsumTest` and put that
directly in this new folder. I'm putting it *directly* in `tests/` because the
`KnpUIpsum` class itself lives directly in `src/`.

And the test file is now gone from the app.

But really... we shouldn't need to update much... or *anything* in the test class
itself. In fact, the *only* thing we need to change is the namespace. Instead of
`App\Tests\Services`, start with the same namespace as the rest of the bundle. So,
`KnpU\LoremIpsumBundle\Tests`.

[[[ code('892e58f7f5') ]]]

But, if we're going to start putting classes in the `tests/` directory, we need to
make sure that Composer can autoload these files. This isn't strictly required to
make PHPUnit work, but it *will* be needed if you add any helper or dummy classes
to the directory and want to use them in your tests.

And, it's easy! We basically want to add a second `PSR-4` rule that says that
the `KnpU\LoremIpsumBundle\Tests` namespace lives in the `tests/` directory. But...
don't! Instead, copy the entire section, paste and rename it to `autoload-dev`.
Change the namespace to end in `Tests\\` and point this at the `tests/` directory.

[[[ code('2beddf5636') ]]]

Why `autoload-dev`? The issue is that our end users will *not* be using anything
in the `tests/` directory: this config exists *just* to help us when we are working
directly on the bundle. By putting it in `autoload-dev`, the autoload rules for
the `tests/` directory will *not* be added to the autoload matrix of our users'
applications, which will give them a slight performance boost.

## Installing symfony/phpunit-bridge

Ok: our test is ready. So let's run it! Move over to the terminal for the
bundle and run... uh... wait a second. Run, what? We haven't installed PHPUnit!
Heck, we don't even have a `vendor/` directory yet. Sure, you *can* run
`composer install` to get a `vendor/` directory... but with nothing inside.

This should be no surprise: if we want to test our bundle, the bundle *itself*
needs to require PHPUnit. Go back to the terminal and run:

```terminal
composer require symfony/phpunit-bridge --dev
```

Two important things. First, we're using Symfony's PHPUnit bridge because it has
a few extra features... and ultimately uses PHPUnit behind-the-scenes. Second, just
like with autoloading, our end users do *not* need to have `symfony/phpunit-bridge`
installed in *their* vendor directory. We *only* need this when we're working on
the bundle itself. By adding it to `require-dev`, when a user installs our bundle,
it will not *also* install `symfony/phpunit-bridge`.

## Ignoring composer.lock

Now that we've run `composer install`, we have a `composer.lock` file! So, commit
it! Wait, don't! Libraries and bundles should actually *not* commit this file -
there's just no purpose to lock the dependencies: it doesn't affect our end-users
at all. Instead, open the `.gitignore` file and ignore `composer.lock`. Now when
we run `git status`, yep! It's gone.

[[[ code('2b038d46e4') ]]]

## phpunit.xml.dist

Ok, let's *finally* run the tests!

```terminal
./vendor/bin/simple-phpunit
```

It - of course - downloads PHPUnit behind the scenes the first time and then...
nothing! It... just prints out the options??? What the heck? Well... our bundle
doesn't have a `phpunit.xml.dist` file yet... so it has *no* idea *where* our
test files live or anything else!

A good `phpunit.xml.dist` file is pretty simple... and I usually steal one from
a bundle I trust. For example, Go to
[github.com/knpuniversity/oauth2-client-bundle](https://github.com/knpuniversity/oauth2-client-bundle).
Find the `phpunit.xml.dist` file, view the raw version and copy it. Back at our
bundle, create that file and paste it in.

[[[ code('ced2b92cdd') ]]]

Oh, and before I forget, in `.gitignore`, *also* ignore `phpunit.xml`. The `.dist`
version *is* committed, but this allows anyone to have a custom version on their
local copy that they do not commit.

[[[ code('4332866cbf') ]]]

Check out the new file: the really important thing is that we set the `bootstrap`
key to `vendor/autoload.php` so that we get Composer's autoloading. This also sets
a few `php.ini` settings and... yes: we tell PHPUnit *where* our test files live.

*Now* I think it *will* work. Find your terminal and try it again:

```terminal-silent
./vendor/bin/simple-phpunit
```

It passes! Woo!

After seeing these fancy green colors, you *might* be thinking that our bundle
is working! And if you did... you'd be *half* right. Next, we'll build a
functional test... which is *totally* going to fail.

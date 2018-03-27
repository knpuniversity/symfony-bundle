# Proper Bundle composer.json File

We put the bundle into our app temporarily because it made it *really* easy
to hack on the bundle, test in the app and repeat.

But now that it's getting kinda stable, it's time to move the bundle into its own
directory with its *own* repository. It's like watching your kid grow up, and *finally*
move into their own apartment.

Find your terminal, and kick that lazy bundle out of your house and into a new directory
next door:

```terminal-silent
mv lib/LoremIpsumeBundle ../LoremIpsumBundle
```

In PhpStorm, let's open that second directory inside a new window, and re-decorate
things a little bit. Ok, a lot to keep track of: application code, bundle code
and terminal. To confuse things more, open a *third* terminal tab and move it
into the bundle, which, sadly, does *not* have a git repository yet!

Let's add one!

```terminal
git init
git status
```

Add everything and commit!

```terminal-silent
git add .
git commit -m "Unicorns"
```

## Bootstrapping composer.json

To make this a shareable package, it needs its very-own `composer.json` file. To
create it, run:

```terminal
composer init
```

Let's call it `knpuniversity/lorem-ipsum-bundle`, give it a description, make sure
the author is correct, leave minimum-stability alone and, for "Package Type" - this
is important! - use `symfony-bundle`. That's needed so that Flex will automatically
enable the bundle when it's installed. For License, I'll use MIT - but more on
that later. And finally, let's *not* add any dependencies yet. And, generate!
Let's *definitely* ignore the `vendor/` directory.

Hello `.gitignore` file and hello `composer.json`! This file has a few purposes.
First, of course, it's where we will eventually require any packages the bundle needs.
We'll do that later. But I am going to start at least by saying that we require php
7.1.3. That's the version that Symfony 4.0 requires.

## Autoloading Rules

Second, the `composer.json` file is where *we* define our autoloading rules: Composer
needs to know what namespace our bundle uses and where those classes live.

Up until now, we put those autoload rules inside the main project. Let's steal that
section and remove the line for our bundle. Paste that into the bundle and remove
the `App` line. The `KnpU\\LoremIpsumBundle\\` namespace lives in just, `src/`.

## Using a "path" Repository

So... yay! We have a standalone bundle with its own repository! But, I'm not
*quite* ready to push this to Packagist yet... and I kinda want to keep testing
it inside my app. But, how? We can't `composer require` it until it lives on
Packagist, right?

Well, there *is* one trick. Google for "composer path package".

Click on the "Repositories" documentation and... *all* the way at the bottom...
there's a `path` option! This allows us to point to any directory on our computer
that contains a `composer.json` file. Then, suddenly, *that* library becomes available
to `composer require`.

Copy the `repositories` section, find our application's `composer.json` and, at
the bottom, paste this. The library lives at `../LoremIpsumBundle`.

Thanks to that, our application *now* knows that there is a package called
`knpuniversity/lorem-ipsum-bundle` available. Back at the terminal, find the tab
for our application and
`composer require knpuniversity/lorem-ipsum-bundle`, with a `:*@dev` at the end.

```terminal-silent
composer require knpuniversity/lorem-ipsum-bundle:*@dev
```

A `path` package isn't quite as smart as a normal package: you don't have versions
or anything like that: it just uses whatever code is in that directory. This tells
Composer to require that package, but not worry about the version.

And, cool! On my system, it installed with a symlink, which means we can keep
hacking on the bundle and testing it live in the app.

Oh, and since Symfony flex noticed that our package has a `symfony-bundle` type,
it actually tried to configure a recipe, which would normally enable the bundle for
us in `bundles.php`. It didn't this time, only because we already have that code.

Now that everything is reconnected, it should work! Refresh the page. Yes! That
bundle is properly living on its own.

Next, we actually already have some tests for our bundle... but they still live
in the app. Let's move these into the bundle and start talking about properly adding
the dependencies that it needs.

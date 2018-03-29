# CI with Travi sCI

Our bundle is missing only two things: it needs a stable release and it needs
continuous integration.

Before we automate our tests, we should probably make sure they still pass:

```terminal
./vendor/bin/simple-phpunit
```

Bah! Boo Ryan: I let our tests get a bit out-of-date. The first failure is in
`FunctionalTest.php` in `testServiceWiringWithConfiguration()`.

Of course: we're testing the `word_provider` option, but that doesn't even exist
anymore! We *could* update this test for the tag system, but it's a little tricky
due to the randomness of the classes. To keep us moving, just delete the test. Also
delete the configuration we added in the kernel, and the `loadFromExtension()` call.
But, just for the heck of it, I'll keep the custom word provider and tag it to
integrate our stub word list.

The *second* failure is in `KnpUIpsumTest`. Ah yea, the first argument to `KnpUIpsum`
is now an *array*. Wrap the argument in square brackets, then fix it in all three
places.

Ok, try the tests again!

```terminal-silent
./vendor/bin/simple-phpunit
```

Yes! They pass.

## Adding the .travis.yml File

The *standard* for continuous integration of open source libraries is definitely
Travis CI. And if you go back to the "Best Practices" docs for bundles, near the
top, Symfony has an *example* of a robust Travis configuration file! Awesome!

Copy this *entire* thing, go back to the bundle, and, at the root, create a new
file - `.travis.yml`. Paste!

We'll talk about some of the specifics of this file in a minute. But first, in
your terminal, add everything we've been working on, commit, and push.

## Activating Travis CI

With the Travis config file in place, the next step is to activate CI for the
repo. Go to [travis-ci.org](https://travis-ci.org/) and make sure you're signed
in with GitHub. Click the "+" to add a new repository, I'll select the "KnpUniversity"
organization and search for lorem.

Huh. Not found. Because it's a new repository, it probably doesn't see it yet.
Click the "Sync Account" button to fix that. And... search again. There it is! If
it's *still* not there for you, keep trying "Sync Account": sometimes, it takes
several tries.

Activate the repo, then click to view it. To trigger the first build, under
"More options", click, ah, "Trigger build"! You don't need to fill in any info
on the modal.

Oh, and from now on, a new build will happen automatically whenever you push. We
only need to trigger the *first* build manually. And... go go go!

## Adjusting PHP & Symfony Version Support

While this is working, let's go look at the `.travis.yml` file. It's... well...
*super* robust: it tests the library on multiple PHP version, uses special flags
to test with the *lowest* version of your library's dependencies and even tests against
multiple versions of Symfony. Honestly, it's a bit ugly, but the result is impressive.

Back on Travis CI, uh oh, we're starting to see failures! No! Let's click on one
of them. Interesting... it's some PHP version issue! Remember, we decided to support
only PHP 7.1.3 or higher. But... we're testing the bundle against PHP 7.0! We
*could* allow PHP 7.0... but let's stay with 7.1.3. In the Travis matrix, delete
the 7.0 test, and change the `--prefer-lowest` to use 7.1.

Go back to the main Travis page again. Hmm: two failures at the bottom deal with
something called `symfony/lts`. These make sure that Symfony works with the LTS -
long-term support version - of Symfony 2 - so Symfony 2.8 - as well as the LTS of
version 3 - so Symfony 3.4. Click into the LTS version 3 build. Ah, it can't install
the packages: `symfony/lts` v3 conflicts with `symfony/http-kernel` version 4.

The test is trying to install version *3* of our Symfony dependencies, but that
doesn't work, because *our* bundle requries everything at version 4!

And... that's *maybe* ok! If we *only* want to support Symfony 4, we can just delete
that test. But I think we should *at least* support Symfony 3.4 - the latest LTS.

To do that, in `composer.json`, change the version to `^3.4 || ^4.0`. Use this for
*all* of our Symfony libraries.

The cool thing is, we don't *actually* know whether or not our bundle *works* with
Symfony 3.4. But... we don't care! The tests will tell us if there are any problems.

Also, in `.travis.yml`, remove the lts v2 test.

Ok, find your terminal, add, commit with a message, and... push!

This should immediately trigger a build. Click "Current"... there it is!

Let's fast-forward... they're starting to pass... and... cool! The first 5 pass!
The last one is still running and, actually, that's going to fail! But don't worry
about it: this is testing our bundle agains the latest, unreleased version of Symfony,
so we don't care too much if it fails. But, I'll show you why it's failing in a minute.

## Tagging Version 1.0

Now that our tests are passing - woo! - it's time to tag our first, official release.
You can do this from the command line, but I kinda like the GitHub interface. Set
the version to `v1.0.0`, give it a title, and describe the release. This is where
I'd normally include more details about new features or bugs we fixed. Then, publish!

You can also do pre-releases, which is a good idea if you don't want to create a
stable version 1.0.0 immediately. On Packagist, the release *should* show up here
automatically. But, I'm impatient, so click "Update" and... yes! There's our version
1.0.0!

Oh, before I forget, back on Travis, go to "Build History", click master and, as
promised, the last one failed. I just want to show you *why*: it failed because of
a deprecation warning:

> Referencing controllers with a single colon is deprecated in Symfony 4.1.

Starting in Symfony 4.1, you should refer to your controllers with *two* colons
in your route. To stay compatible with 4.0, we'll leave it.

## Installing the Stable Release

Now that we *finally* have a stable release, let's install it in our app. At
your terminal, first remove the bundle:

```terminal
composer remove knpuniversity/lorem-ipsum-bundle
```

Wait.... then re-add it:

```terminal
composer require knpuniversity/lorem-ipsum-bundle
```

Yes! It got v1.0.0.

We have an awesome bundle! It's tested, it's extendable, it's on GitHub, it has
continuous integration, it can bake you a cake and it has a stable release.

I hope you learned a ton about creating re-usable bundles... and even more about
how Symfony works in general. As always, if you have any questions or comments,
talk to us down in the comments section.

All right guys, seeya next time.

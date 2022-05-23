# Publishing to Packagist

Our bundle is ready to be shared with the world! So let's take care of a few last
details, and publish our bundle to Packagist!

## Choosing a License

But, before we publish this *anywhere*, we need do some boring, but very important
legal work. Go to [choosealicense.com](https://choosealicense.com) and find the
license that works best for you. Symfony is licensed MIT, and that's *definitely*
the best practice. Whatever you choose, copy the license, find your bundle code,
and at the root, create the `LICENSE` file.

[[[ code('68152f15fb') ]]]

## Pushing to GitHub

Legal stuff, done! Next, find your terminal: there are a bunch of uncommitted changes.
Oh, before we add them, I made a mistake!

I have an extra `tests/Controller/cache` directory! Open `IpsumApiControllerTest`
and find the `getCacheDir()` method. I *meant* to change this to use the same cache
directory as `FunctionalTest`, which is already set to be ignored by git. Add a
`../` to the path. Then, delete the extra `cache/` dir. There's also  an extra
`logs` directory, but it's empty, so just ignore it.

[[[ code('f69bbd52b4') ]]]

*Now* move back to your terminal, add everything to git, give it an inspiring message,
and commit!

With everything committed, let's push this to GitHub! Well, you can host it *anywhere*,
but GitHub is the most common place. I'll click "New Repository", choose the
KnpUniversity organization, and name it `lorem-ipsum-bundle`.

It's not *required*, but it's usually nice to name the repository the same as the
package name in `composer.json`. Give it a clever description, make sure it's public,
and create repository!

Copy the code to push to an existing repository, go find your terminal, quick!
Paste, hit enter, wait impatiently... then... say hello to our new repository!

## Registering on Packagist

With that done, we can *now* put our bundle up on Packagist! Go to
[packagist.org](https://packagist.org/) and make sure you're logged in. Then, it's
*super* easy: click "Submit", copy the GitHub URL, paste, and click "Check".

This does some sanity checks in the background, like parsing your `composer.json`
file and waiting for Jordi to search for any similar packages on Packagist, to help
avoid duplication.

Looks ok! Moment of truth: Submit!

Boom! We are a package!

## Auto-updating with the GitHub Service Hook

Oh, but notice this message:

> The package is not auto-updated. Please setup the Github Service Hook

This is actually important. When we create a new tag in GitHub, we want Packagist
to automatically see it.

Go back to GitHub, click Settings, Integration & services, "Add service" and find
Packagist. You'll need to enter your username and a token you can find on your
Packagist profile page. Then, add service!

## Requiring the new Package

And, for now, we're done! We have a *real* package! Next, open our application's
`composer.json` file. We're still using this `path` repository option. Let's
*finally* install our package *properly*. Remove the `repositories` section.

Then, go to the terminal for your app, and, first, *remove* the current package:

```terminal skip-ci
composer remove knpuniversity/lorem-ipsum-bundle
```

Gone! And thanks to the Flex recipe, it also removed the bundle from `bundles.php`.
Cool!

Now, lets re-install it:

```terminal
composer require knpuniversity/lorem-ipsum-bundle
```

This downloads `dev-master`, so the `master` branch, because there's no tag yet.
*And*! Flex re-added the bundle to `bundles.php`.

## Writing a Decent README

Cool! But, go back to the GitHub page for our bundle. See anything missing? Yea,
no README! That's not ok! If you go back to the "Symfony bundle best practices"
page, this has an example README you can use to get started.

Head back to our code, I'll close a few files, then create a new `README.md` file.
And, bam! I just wrote us a README file!

[[[ code('19216c7c7d') ]]]

Don't worry, I'm not going to lecture you on how to write README files. Well, actually,
can I take just *one* minute to point out the *most* important parts that I think
people sometimes forget?

To start, make sure your bundle has these four parts. One, at the top, say what
the bundle does in plain language! Two, show the `composer require` installation
command. Three, give a simple usage example, before talking about any other technical
jargon. And four, show the configuration.

After that, you can talk about whatever complex or theoretical stuff you want, like
how to create a word provider.

Also, when you create code examples, there are *two* common mistakes. First, make
sure you include the file path as a comment: people don't always know where a file
should live. Second, *don't* create the code blocks here. Believe me, you'll make
a mistake. Code them in a *real* app, paste them here, then tweak.

Oh, and for the configuration section, remember, you can run:

```terminal
php bin/console config:dump knpu_lorem_ipsum
```

to get a *full* config tree to paste here. Oh, and, if the user needs to *create*
a file - like `knpu_lorem_ipsum.yaml`, say that explicitly: sometimes people think
they're doing something wrong if a file doesn't already exist.

## A Recipe?

The *last* thing I would recommend is, if it makes sense, create a recipe for your
bundle. Do this at [github.com/symfony/recipes-contrib](https://github.com/symfony/recipes-contrib).
We're not going to do this, but if your bundle needs a config file or *any* other
setup, this is a *huge* way to make it easier to use.

If you *don't* create a recipe, Flex will at least enable the bundle automatically.
And in a lot of cases - like for this bundle - that's enough.

Ok, just *one* topic left, and it's fun! Let's setup continuous integration on
Travis CI so that we can be sure our tests are always passing.

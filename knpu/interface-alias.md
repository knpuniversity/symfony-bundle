# Extensibility with Interfaces & Aliases

I want to make two other changes to the new "word provider" setup. The first is
optional: it's another common method for making the word provider configurable.

Go back into our `services.xml` file. Right now, we set the first argument inside
of the XML file, then override that argument in the extension class, if a different
value is provided. Another option - and we'll talk about the advantages later - is
to use a service *alias*.

Copy the alias we created earlier in order to enable autowiring. Create a new alias
whose id is `knpu_lorem_ipsum.word_provider` and set the alias to the
`knp_word_provider` service id above.

[[[ code('6d4cc8dcdc') ]]]

Thanks to this, there is now a *new* service in the container called
`knpu_lorem_ipsum.word_provider`. But when someone references it, it actually
just points to our `knpu_lorem_ipsum.knpu_word_provider`. Now, for the argument
to `KnpUIpsum`, pass the *alias* id instead.

[[[ code('5bc6d1f4d6') ]]]

So far, this won't change *anything*. But open the extension class. Instead of changing
the argument, we can override the *alias* to point to *their* service id. Do this
with `$container->setAlias()`. First pass `knpu_lorem_ipsum.word_provider`
and set this alias to `$config['word_provider']`. We don't need the `new Reference()`
here because the `setAlias()` method expects this to be a service ID.

[[[ code('916cc1af03') ]]]

And before even trying it, copy the service alias, find your terminal, and run:

```terminal
php bin/console debug:container --show-private knpu_lorem_ipsum.word_provider
```

Yes! This is an alias to our `CustomWordProvider`. And *that* means that the
first argument to `KnpUIpsum` will use that. Refresh to make sure it still works.
It does!

There's no *amazing* reason to use this alias strategy versus what we had before,
but there are two minor advantages. First, *if* we needed to reference the word
provider service in multiple places - probably in `services.xml` - using an alias
is *easier*, because you don't need to remember to, for example, replace 5 different
arguments where the service is used. And second, *if* we wanted this service to be
used *directly* by our users, creating an alias is the *only* way to give them a
service id they can reference, *even* if they override the word provider to be something
else.

## Creating a WordProviderInterface

Ok, our setup is really, really nice. But there is *one* restriction we're putting
on our user that we really do *not* need to! Open `KnpUIpsum` and scroll all the
way to the constructor. The first argument is type-hinted with `KnpUWordProvider`.
This means that if the user wants to create their *own* word provider, they *must*
extend our *original* `KnpUWordProvider`. We *are* doing this... because we just
want to add a new word to the list, but this should *not* be required! All *we*
care about is that the service has a `getWordList()` method that returns an array.

In other words, this is the *perfect* use-case for an interface! Wooo! In the bundle,
create a new PHP class. Call it `WordProviderInterface` and change the "kind" from
class to interface.

Inside, add the `getWordList()` method and make it return an array. This is *also*
the perfect place to add some documentation about what this method should do.

[[[ code('4483124629') ]]]

With the interface done, go back to `KnpUIpsum`, change the type-hint to
`WordProviderInterface`. The user can now pass *anything* they want, as long as it
has this `getWordList()` method... because *that* is what we're using at the bottom
of `KnpUIpsum`.

[[[ code('b8ea260532') ]]]

Then, of course, we also need to go open *our* provider and make sure it implements
this interface: `implements WordProviderInterface`.

[[[ code('6a48385ce2') ]]]

If you try it now... *not* broken! And yea, *our* `CustomUserProvider` will *still*
extend `KnpUWordProvider`, but that's now optional - we could just implement the
interface directly.

Next, let's take a big step and move our bundle *out* of our code and give it it's
*own* `composer.json` file!

# Tags, Compiler Passes & Other Nerdery

Let's review: we gave our service a tag. And now, we want to tell Symfony to
find *all* services in the container with this tag, and pass them as the first
argument to our `KnpUIpsum` service. Like I mentioned in the previous chapter, if
you only need to support Symfony 3.4 or higher, there's a shortcut. But if you
need to support lower versions or want to geek out with me about compiler passes,
well, you're in luck!

First question: how can we find *all* services that have the
`knpu_ipsum_word_provider`
tag? If you look in the extension class, you might think that we could do some
magic here with the `$container` variable. And... yea! It even has a method called
`findTaggedServiceIds()`!

But... you actually *can't* do this logic here. Why? Well, when this method is called,
not *all* of the other bundles and extensions have been loaded yet. So if you tried
to find all the services with a certain tag, some of the services might not be in
the container yet. And actually, you can't even get *that* far: the `ContainerBuilder`
is *empty* at the beginning of this method: it doesn't contain *any* of the services
from *any* other bundles. Symfony passes us an empty container builder, and then
merges it into the *real* one later.

## Compiler Pass

The *correct* place for any logic that needs to operate on the *entire* container,
is a compiler pass. In the `DependencyInjection` directory - though it doesn't
technically matter where this class goes - create a `Compiler` directory then a
new class called `WordProviderCompilerPass`. Make this, implement a
`CompilerPassInterface`, and then go to the Code -> Generate menu - or
Command + N on a Mac - click "Implement Methods" and select `process()`.

A compiler pass *also* receives a `ContainerBuilder` argument. But, instead of being
empty, this is full of *all* of the services from *all* of the bundles. That means
that we can say `foreach ($container->findTaggedServiceIds()`, pass this the
tag we're using: `knpu_ipsum_word_provider`, and say `as $id => $tags`.

This is a little confusing: the `$id` key is the service ID that was tagged. Then,
`$tags` is an array with extra information about the tag. Sometimes, a tag can
have other attributes, like priority. You can also tag the same service with the
same *tag*, multiple times.

Anyways, we don't need that info: let's just `var_dump($id)` to see if it works,
then die.

## Registering the Compiler Pass

To *tell* Symfony about the compiler pass, open your *bundle* class. Here, go back
to the Code -> Generate menu - or Command + N on a Mac - choose "Override Methods"
and select `build()`. You don't need to call the parent `build()` method: it's
empty. *All* we need here is `$container->addCompilerPass(new WordProviderCompilerPass())`.

There are different *types* of compiler passes, which determine when they are executed
relative to *other* passes. And, there's also a priority. But unless you're doing
something *really* fancy, the standard type and priority work fine.

Thanks to this line, whenever the container is built, it *should* hit our die statement.
Let's move over to the browser and, refresh!

Yes! There is the *one* service that has the tag.

And now... it's easy! The code in a compiler pass looks a lot like the code in an
extension class. At the top, add
`$definition = $container->getDefinition('knpu_lorem_ipsum.knpu_ipsum')`.

Ultimately, we need to modify *this* services's first argument. Create an empty
`$references` array. And, in the foreach, just add stuff to it:
`$references[] = new Reference()` and pass in the `$id`.

Finish this with `$definition->setArgument()`, pass it `0` for the first argument,
and the array of reference objects.

We're done! Go back to our browser and try it! Woohoo! We're now passing an *array*
of all of the word provider services into the `KnpUIpsum` class.... which... yea,
is just one right now.

## Cleanup the Old Configuration

With this in place, we can remove our old config option. In the `Configuration`
class, delete the `word_provider` option. And in the extension class, remove the
code that reads this.

## Tagging the CustomWordProvider

Next, move over to the application code, and in `config/packages/knpu_lorem_ipsum.yaml`,
yep, take out the `word_provider` key.

If you refresh now... it's going to work. But, not surprisingly, the word "beach"
will not appear in the text. Remember: "beach" is the word that we're adding with
our `CustomWordProvider`. This class is *not* being used. And... that make sense!
We haven't tagged this service with *anything*, so our bundle doesn't know to use
it.

Before we do that, now that there are *multiple* providers, I don't need to extend
the core provider anymore. Implement the `WordProviderInterface` directly. Then,
just return an array with the one word: `beach`.

To tag the service, open `config/services.yaml`. This class is automatically
registered as a service. But to give it a tag, we need to override that:
`App\Service\CustomWordProvider`, and, below, `tags: [knpu_ipsum_word_provider]`.

Let's try it! Refresh! Yes! It's alive!

## Setting up Autoconfiguration

But... there's something that's bothering me. *Most* of the time in Symfony, you
*don't* need to manually configure the tag. For example, earlier, when we created
an event subscriber, we did *not* need to give it the `kernel.event_subscriber` tag.
Instead, Symfony was smart enough to see that our class implemented
`EventSubscriberInterface`, and so it added that tag for us *automatically*.

So... what's the difference? Why can't the tag be automatically added in *this*
situation? Well... it can! But we need to set this up in our bundle. Open the extension
class, go anywhere in the `load()` method, and add
`$container->registerForAutoconfiguration(WordProviderInterface::class)`. The feature
that automatically adds tags is called autoconfiguration, and this method returns
a "template" `Definition` object that we can modify. Use
`->addTag('knpu_ipsum_word_provider')`.

Cool, right? Back in our app code, remove the service entirely. And now, try it!
Hmm, no beach the first time but on the second refresh... we got it!

We now have a *true* word provider plugin system. *And* creating a custom word provider
is as easy as creating a class that implements `WordProviderInterface`.

Next, let's finally put our library up on Packagist!

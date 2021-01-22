# Bundle Configuration Class

The `KnpUIpsum` class *has* two constructor args, but the user can't control these...
yet. In `knpu_lorem_ipsum.yaml`, here's my idea: allow the user to use two new
config keys, like `unicorns_are_real` and `min_sunshine`, and pass those values
to our service as arguments.

Comment-out the `var_dump`. Symfony's configuration system is *smart*: all the
keys are *validated*. If you typo a key - like `secret2` under `framework`, when
you refresh, you get a big ol' error! Yep, *each* bundle creates its own "tree" of
*all* the valid config keys.

In fact, find your terminal. Run:

```terminal
php bin/console config:dump framework
```

This is an example of the *entire* tree of valid configuration for `framework`!
This is *amazing*, and it's made possible by a special `Configuration` class. It's
time to create our own!

## Creating the Configuration Class

Inside the `DependencyInjection` directory, create a new class called `Configuration`.
Make this implement `ConfigurationInterface`: the one from the `Config` component.
We'll need to implement one method: go to the Code -> Generate menu, or `Cmd`+`N` on
a Mac, select "Implement Methods" and choose `getConfigTreeBuilder()`.

[[[ code('bbcfe9a420') ]]]

This is one of the *strangest* classes you'll ever see. By using PHP code, we're
going to define the entire *tree* of valid config that can be passed to our bundle.

A *great* way to see how this class works is to look at an existing one! Type
Shift+Shift to open a class called `FrameworkExtension`, deep in the core of Symfony.
Yep, this is the extension class for FrameworkBundle! It has the same `load()`
method as *our* extension.

In the same directory, if you click on the top tree, you'll find a class called
`Configuration`. Inside, it defines *all* of the valid config keys with a, sort of,
nested tree format. This is a super powerful and, honestly, super complex system.
We're only going to use a few basic features. If you need to define a more complex
config tree, *definitely* steal, um, borrow, from these core classes.

## Building the Config Tree

Back in *our* class, start with `$treeBuilder = new TreeBuilder()`. Then,
`$rootNode = $treeBuilder->root()` and pass the name of our key:
`knpu_lorem_ipsum`.

[[[ code('f8fb0443d9') ]]]

***TIP
Since Symfony 4.3 you should pass the root node name to the `TreeBuilder` instead:

```php
$treeBuilder = new TreeBuilder('knpu_lorem_ipsum');
$rootNode = $treeBuilder->getRootNode();
// ...
```
***

Now... just start building the config tree! `$rootNode->children()`, and below,
let's create two keys. The first will be for the "unicorns are real" value,
and it should be a boolean. To add that, say `->booleanNode('unicorns_are_real')`,
`->defaultTrue()` and to finish configuring this node, `->end()`.

[[[ code('3c45ce51e5') ]]]

The other option will an integer: `->integerNode('min_sunshine')`, default it to 3,
then `->end()`. Call `->end()` one more time to finish the `children()`.

[[[ code('a574871f09') ]]]

Weird, right!? Return the `$treeBuilder` at the bottom.

[[[ code('c771c856ae') ]]]

## Using the Configuration Class

In our extension, we can use this to validate and merge all the config together.
Start with `$configuration = $this->getConfiguration()` and pass this `$configs`
and the container. This simply instantiates the `Configuration` class.

[[[ code('3a11cce9f0') ]]]

Here's the *really* important part: `$config = $this->processConfiguration()`: pass
the configuration object and the original, raw array of `$configs`. `var_dump()`
that final config and `die`!

[[[ code('ad03ae489e') ]]]

Let's see what happens! Find your browser and... refresh! We get an error... which
is awesome! It says:

> Unrecognized option "bar" under "knpu_lorem_ipsum"

This is telling us:

> Yo! "bar" is not one of the valid config keys!

Back in `knpu_lorem_ipsum.yaml`, temporarily comment-out *all* of our config.
And, refresh again. Yes! No error! Instead, we see the final, validated & normalized
config, with the *two* keys we created in the `Configuration` class.

[[[ code('2852324fbb') ]]]

Put *back* the config, but use a real value: `min_sunshine` set to 5.

[[[ code('8e8b2c3063') ]]]

Refresh one last time. Woohoo! `min_sunshine` equals 5. These `Configuration`
classes are strange... but they take care of everything: validating, merging and
applying default values.

## Dynamically Setting the Arguments

We are *finally* ready to *use* this config. But... how? The service & its arguments
are defined in `services.xml`... so we can't just magically reference those dynamic
config values here.

Copy the service id and go back to the extension class. That container builder holds
the *instructions* on how to instantiate our service - like its class and what constructor
arguments to pass to it. And we - right here in PHP - can *change* those.

Check it out: start with `$definition = $container->getDefinition()` and pass the
service id. This returns a `Definition` object, which holds the service's class
name, arguments and a bunch of other stuff. *Now* we can say
`$definition->setArgument()`: set the first argument - which is index 0 - to
`$config['']`. The first argument is `$unicornsAreReal`. So use the
`unicorns_are_real` key. Set the second argument - index one - to `min_sunshine`.

[[[ code('037de01112') ]]]

That's it! Go back and refresh! It works! Sunshine now appears at least 5 times
in every paragraph. Our dynamic value *is* being passed!

Oh, and, bonus! In your terminal, run `config:dump` again, but *this* time pass
it `knpu_lorem_ipsum`:

```terminal-silent
php bin/console config:dump knpu_lorem_ipsum
```

Yes! Our bundle now prints its config thanks to the `Configuration` class. If you
want to get *really* fancy - which of course we *do* - you can add documentation
there as well. Add `->info()` and pass a short description about why you would
set this. Do the same for `min_sunshine`.

[[[ code('37d7ee8fb4') ]]]

Run `config:dump` again:

```terminal-silent
php bin/console config:dump knpu_lorem_ipsum
```

Pretty, freakin' cool.

Next, let's get fancier with our config and allow entire *services* to be swapped
out.

# Allowing Entire Services to be Overridden

When you create a reusable library, you gotta think about what *extension* points
you want to offer your users. Right now, the user can control the two arguments to
this  class... but they can't control anything else, like the actual *words* that
are used in our fake text. These are hardcoded at the bottom.

So... how *could* we allow the user to *override* these? One option that I like is
to *extract* this code into its own class, and allow the user to *override* that
class *entirely*.

Check this out: in the bundle, create a new class called `KnpUWordProvider`.
Give it a public function called `getWordList()` that will return an array. Back
in `KnpUIpsum`, steal the big word list array and... return that from the new
function.

[[[ code('2dfa717a5a') ]]]

Perfect! In `KnpUIpsum`, add a new constructor argument and type-hint it with
`KnpUWordProvider`. Make it the first argument, because it's required. Create a new
property for this - `$wordProvider` - then set it below:
`$this->wordProvider = $wordProvider`.

[[[ code('ecd2338ea8') ]]]

With all that setup, down below in the original method, just return
`$this->wordProvider->getWordList()`.

[[[ code('978250b55a') ]]]

Our class is now *more* flexible than before. Of course, in `services.xml`, we
need to tell Symfony to pass in that new argument! Copy the existing service
node so that we can register the new provider as a service first. Call this one
`knpu_lorem_ipsum.knpu_word_provider` and set the class to `KnpUWordProvider`.
Oh, but this service does *not* need to be public: no one should need to use this
service directly.

[[[ code('87d5ea2b89') ]]]

Above, we need to *stop* using the short service syntax. Instead, add a closing
service tag. Then, add an argument with `type="service"` and
`id="knpu_lorem_ipsum.knpu_word_provider"`.

[[[ code('a28caa7bd3') ]]]

If you're used to configuring services in YAML, the `type="service"` is equivalent
to putting an `@` symbol before the service id. The *last* change we need to make
is in the extension class. These are now the second and third arguments, so use
the indexes one and two.

[[[ code('261a1e8eee') ]]]

Phew! Unless we messed something up, it should work! Try it! Yes! We *still*
get fresh words each time.

## Making the Word Provider Configurable

So... we refactored our code to be more flexible... but it's *still* not possible
for the user to override the word provider. Here's my idea: in the `Configuration`
class, add a new *scalar* node - in other words, a *string* option - called `word_provider`.
Default this to `null`, and you can add some documentation to be *super* cool.
If the user wants to customize the word list, they will set this to the service
*id* of their *own* word provider.

[[[ code('2a35b342c9') ]]]

So, in the extension class, if the that value is *not* set to null, let's *replace*
the first argument entirely: `$definition->setArgument()` with 0 and
`$config['word_provider']`.

[[[ code('ec7a835563') ]]]

## Creating our Custom Word Provider

We're *not* setting this config value yet, but when we refresh, great! We at least
didn't *break* anything... though we *do* have a small mistake...

Anyways, let's test the system properly by creating our own, new word provider.
In `src/Service`, create a class called `CustomWordProvider`. Make this extend
the `KnpUWordProvider` because I just want to *add* something to the core list.
To override the method, go to the Code -> Generate menu, or Cmd+N on a Mac - choose
"Override methods" and select `getWordList()`.

[[[ code('04337b4972') ]]]

Inside, set `$words = parent::getWordList()`. Then, add the word "beach"... because
we all deserve a little bit more beach in our lives. Return `$words` at the bottom.

[[[ code('73660af444') ]]]

Thanks to the standard service configuration in our app, this class is already
registered as a service. So all *we* need to do is go into the `config/packages`
directory, open `knpu_lorem_ipsum.yaml`, and set `word_provider` to
`App\Service\CustomWordProvider`.

[[[ code('7cb66d1000') ]]]

Let's see if this thing works! Move over and refresh! Boooo!

> Argument 1 passed to KnpUIpsum::__construct() must be an instance of
> KnpUWordProvider - because that's our type-hint - *string* given.

Look below in the stack-trace: this is pretty deep code, but you can actually
see that something is creating a new `KnpUIpsum`, but passing the string *class*
name of our provider as the first argument... not the service!

Go back to our extension class. Here's the fix: when we set the argument to
`$config['word_provider']`, this *of course* sets that argument to the *string*
value! To fix this in YAML, we would prefix the service id with the `@` symbol.
In PHP, wrap the value in a `new Reference()` object. *This* tells Symfony that
we're referring to a *service*.

[[[ code('5ba8e3b24e') ]]]

Deep breath and, refresh! It works! And if you search for "beach"... yes!
Let's go to the beach!

This is a great step! But there are two other nice improvements we can make: using
a service alias & introducing an interface. Let's add those next.

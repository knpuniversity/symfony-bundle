# All about the Bundle Extension Config System

We're not passing *any* arguments to the service... but this class *does* have two
*very* important arguments: whether or not unicorns are real and the minimum times
the word sunshine should appear in each paragraph. But what if a user of our bundle
wants *more* sunshine or - gasp - they don't believe in unicorns? Right now, there's
*no* way for them to control these arguments.

So if the *bundle* is responsible for registering the services & passing its arguments,
how can the *user* of that bundle *control* those arguments? The answer lives in
the `config/packages` directory.

Some important notes: first, our app automatically loads & processes *all* `.yaml`
files it finds in this directory. Second, the *names* of these files are *not*
important: you could rename them to *anything* else, `.yaml`. And *third*, the
*entire* purpose of these files is to control the services that are provided by
different bundles. When Symfony sees the `framework` key, it passes this configuration
to the FrameworkBundle, which uses it to modify the services it provides.

The same for `monolog`: this config is passed to MonologBundle and it uses that
when registering its services.

## Creating a New Config File

Create a new file: `knpu_lorem_ipsum.yaml` - but, we could call this anything.
And just to see what will happen, add some fake config: `foo:`, then `bar: true`.

Find your browser and refresh! Error! Check out the language carefully. It says
that there is no *extension* able to load the configuration for "foo". We *know*
that word extension: we just created our *own* extension: `KnpULoremIpsumExtension`.

Then, since `foo` is apparently invalid, it lists a bunch of *valid* keys,
like `framework`, `web_server`, `twig`, etc. Here's the deal: when Symfony sees
a root key like `framework`, it looks at *all* of the bundles, well, really, the
*extension* class for each bundle, to see if there is one called `FrameworkExtension`.
If there *is*, it passes the config to it. If there is *not*, it throws this big,
hairy, ugly exception. 

## Passing Config to our Extension

But check this out: go back to the list of valid keys. Thanks to our
`KnpULoremIpsumExtension` class, there's one called `knp_u_lorem_ipsum`! Change
the root key to use *that* instead. Next, open our extension class,
`var_dump($configs)` and die.

Try it out! No error! And cool! That `bar: true` value is passed to the
`load` method! We're one step closer to *using* that config to tweak our service.

But, there are two weird things. First, the root key is... uh... not perfect.
The `knp_u_` is.. weird - I want it be `knpu_`... but apparently Symfony disagrees:
our extra capital "U" is confusing things. We'll fix this in a bit.

The second weird thing is that the `$configs` value that's passed to `load()` is
*not* just a simple array with `bar=true`. Nope, it's an *array* of arrays. Inception.
Why? Well, it's possible that the user could add configuration for our bundle in
*multiple* files. Like, we could have a `dev` environment-specific YAML file. When
that happens, instead of merging that config together, it would pass us the configuration
from *both* files. For example, if `knp_u_lorem_ipsum` existed in *three* different
files, this array would have *three* different arrays inside. And, yep! It will
be our job to merge them together. But, that's actually going to be *really* cool.

But before we do that, let's fix our alias to be `knpu_lorem_ipsum`. It's not something
you *often* need to worry about, but the fix is *super* interesting.

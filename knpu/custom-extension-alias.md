# Custom Extension Alias

When you create an extension class, Symfony automatically calculates a "root" config
key for it. In our case, it calculated `knp_u_lorem_ipsum`... it generated this based
on our class name. I'd *rather* have `knpu_lorem_ipsum`. But of course, that doesn't
work... yet.

This root key is called the extension *alias*. And *we* can *totally* control it.
How? In our extension class, go to the Code->Generate menu, or Cmd+N on a mac, select
"Override" methods, and choose `getAlias()`. Then, return `knpu_lorem_ipsum`.

Here's how things *really* work. When Symfony boots, it loops over all the extension
classes in the system, calls `getAlias()` on each, and *this* becomes their config
key. In the parent class, well, the *parent's* parent class, there is a *default*
`getAlias()` method which... surprise! Removes the `Extension` suffix, and "underscores"
what's left.

Cool! Easy fix! Find your browser and refresh to celebrate! Boo! Another error:

> Users will expect the alias of the default extension of a bundle to be the
> underscored version of the bundle name. You can override some method if you want
> to use another alias.

## How Bundles Load Extensions

Ok. This is a bit odd, but, on the bright side, it'll give us a chance to do some
exploring! Open up our bundle class. It's empty... but it actually does a *bunch*
of cool things. Hold Command or Ctrl and click to open the base class. One of the
methods is called `getContainerExtension()`.

When Symfony builds the container, it loops over all bundle classes and calls this
method, which returns the extension object. Check out the `createContainerExtension()`
method, well, actually, the `getContainerExtensionClass()` method. Ah! *This* is
the reason why Symfony expects our extension to live in the `DependencyInjection`
directory and to end in the word `Extension`. *All* that magic comes from overrideable
methods on our bundle class.

Scroll back up to `getContainerExtension()`. After it creates the container extension,
it does a sanity check: if the alias is different than it expected, it throws an
exception. This was originally added to prevent bundle authors from going *crazy*
and creating custom aliases like `delicious_pizza` or `beam_me_up_scotty`.

But, it's kind of annoying. The fix is easy. In our bundle class, go to the
Code -> Generate menu, or Cmd + N on a Mac, select Override Methods and choose
`getContainerExtension`.

Then, if `null === $this->extension`, set `$this->extension` to a new
`KnpULoremIpsumExtension`. Return `$this->extension` at the bottom.

This does the same thing as the parent method, but without that sanity check.

Let's do it... refresh! Our custom alias is alive!!!

Now, it's time to use this `$configs` array to start allowing our end-users to
modify our service. *This* is one of my *favorite* parts.

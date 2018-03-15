# Auto-Adding Services

At this point... we have a *directory* with a PHP class inside. And, honestly,
we *could* just move this into its own repository, put it on Packagist and be done!
But in that case, it wouldn't be a *bundle*, it would simply be a *library*, which
is more or less defined as: a directory full of PHP classes.

So what *is* the difference between a library and a bundle? What does a bundle give
is that a library does not? The "mostly-accurate" answer is simple: *services*.
If we *only* created a library, people could use our classes, but it would be up
to *them* to add configuration to register them as *services* in Symfony's container.
But if we make a bundle, *we* can *automatically* add services to the container
as *soon* as our bundle is installed. Sure, bundles can also do a few other things -
like provide translations and other config - but providing services is their main
super power.

So, we're going to create a bundle. Actually, the *perfect* solution would be to
create a *library* with only the `KnpUIpsum` class, and then *also* a bundle that
*requires* that library and adds the Symfony service configuration. A good example
of this is KnpMenu and KnpMenuBundle.

## Creating the Bundle Class

To make this a bundle, create a new class called `KnpULoremIpsumBundle`. This could
be called anything... but usually it's the vendor namespace plus the directory
name.

Make this extend `Bundle` and... that's it! You almost *never* need to have any
logic in here.

To enable this in our app, open `bundles.php` and configure it for all environments.
I'll remove the `use` statement for consistency. Normally, this happens automatically
when we install a bundle... but since we just added the bundle manually, we gotta
do it by hand.

And, congratulations! We now have a bundle!

## Creating the Extension Class

So.... what the heck does that give us? Remember: the super-power of a bundle is
that it can *automatically* add services to the container, without the user needing
to configure *anything*. How does that work? Let me show you.

Next to the bundle class, create a new directory called `DependencyInjection`. Then,
add a new class inside with the same name of the bundle, except ending in `Extension`.
So, `KnpULoremIpsumExtension`. Make this extend `Extension` from `HttpKernel`.
This forces us to implement *one* method. I'll go to the Code -> Generate menu,
or Cmd+N on a Mac, choose "Implement Methods" and select the one we need. Inside,
just `var_dump` that we're alive and... die!

*Now* move over and refresh. Yes! It hits our new code!

This is *really* important. Whenever Symfony builds the container, it loops over
all the bundles and, inside of each, looks for a `DependencyInjection` directory
and then inside of that, a class with the same name of the bundle, but ending in
`Extension`. Woh. *If* that class exists, it instantiates it and calls `load()`.
This is *our* big chance to *add* any services we want! We can go *crazy*!

See this `$container` variable? It's not *really* a container, it's a container
*builder*: something we can add services to.

## Adding services.xml

Right now, our service is defined in the `config/services.yaml` file of the application.
Delete that! We're going to put a service configuration file *inside* the bundle
instead. Create a `Resources/` directory and another `config/` directory inside:
this is the best-practice location for service config. Then, add `services.xml`.
Yep, I said *XML*. Wait, don't run away!

You *can* use YAML to configure your services, but XML is the best-practice for
re-usable bundles... though it doesn't matter much. Using XML *does* have one
tiny advantage: it doesn't require the `symfony/yaml` component, which, at least
in theory, makes your bundle feel a bit lighter.

To fill this in... um, I cheat. Google for "Symfony Services", open the documentation,
search for XML, and stop when you find a code block that *defines* a service. Click
the XML tab and steal this! Paste it into our code. The only thing *we* need to
do is configure a single service whose id is the class of the service. So, use
`KnpU\LoremIpsumBundle\KnpUIpsum`. We're not passing any arguments, so we can use
the short XML syntax for now.

But this file isn't processed automatically. Go to the extension class and remove
the `var_dump()`. The code to load the config file looks a little funny:
`$loader = new XmlFileLoader()` from the DependencyInjection component. Pass
this a `new FileLocator` - the one from the `Config` component - with the path
to that directory: `../Resources/config`. Below that, add
`$loader->load('services.xml')`.

Voilà! Refresh the page. It works! When the container builds, the `load()`
method is called and our bundle adds its service.

Next, let's talk about service id best-practices, how to support autowiring and
public versus private services.

# Adding Routes & Controllers

If you watch a lot of KnpU tutorials, you know that I *love* to talk about how the
*whole* point of a bundle is that it adds *services* to the container. But, even
I have to admit that a bundle can do a lot more than that: it can add routes,
controllers, translations, public assets, validation config and a bunch more!

Find your browser and Google for "Symfony bundle best practices". This is a really
nice document that talks about how you're *supposed* to build re-usable bundles.
We're following, um, *most* of the recommendations. It tells you the different
directories where you should put different things. Some of these directories are
just convention, but some are required. For example, if your bundle provides
translations, they need to live in the `Resources/translations` directory next
to the bundle class. If you follow that rule, Symfony will automatically load them.

## Adding a Route + Controller

Here's our *new* goal: I want to add a route & controller to our bundle. We're going
to create an optional API endpoint that returns some delightful lorem ipsum text. 

Before we start, I'll open my PhpStorm preferences and, just to make this more fun,
search for "Symfony" and enable the Symfony plugin. Also search for "Composer" and
select the `composer.json` file so that PhpStorm knows about our autoload namespaces.

Back to work! In `src/`, create a `Controller` directory and inside of that, a
new PHP class called `IpsumApiController`. We don't need to make this extend anything,
but it's OK to extend `AbstractController` to get some shortcuts... except what!?
`AbstractController` doesn't exist!

That's because the class lives in `FrameworkBundle` and... remember! Our bundle
does *not* require that! Ignore this problem for now. Instead, find our app code,
open `AbstractController`, copy its `namespace`, and use it to add the `use` statement
manually to the controller.

[[[ code('7ff55abd7e') ]]]

Next, add a public function called `index`. Here, we're going to use the `KnpUIpsum`
class to return a JSON response with some dummy text. When you create a controller
in a reusable bundle, the best practice is to register your controller as a proper
service and use dependency injection to get anything you need.

[[[ code('d4f87bd706') ]]]

Add `public function __construct()` and type-hint the first argument with `KnpUIpsum`.
I'll press Alt+Enter and choose Initialize Fields so that PhpStorm creates and
sets a property for that.

[[[ code('a19a2ca121') ]]]

Down below, return `$this->json()` - we will *not* have auto-complete for that method
because of the missing `AbstractController` - with a `paragraphs` key set to
`$this->knpUIpsum->getParagraphs()` and a `sentences` key set to
`$this->knpUIpsum->getSentences()`

[[[ code('87a4bfc66f') ]]]

Excellent!

## Registering your Controller as a Service

Next, we need to register this as a service. In `services.xml`, copy the first
service, call this one `ipsum_api_controller`, and set its class name. For now,
*don't* add `public="true"` or `false`: we'll learn more about this in a minute.
Pass one argument: the main `knpu_lorem_ipsum.knpu_ipsum` service.

[[[ code('8cd36159d4') ]]]

***TIP
In Symfony 5, you'll need a bit more config to get your controller service working:

```xml
<service id="knpu_lorem_ipsum.ipsum_api_controller" class="KnpU\LoremIpsumBundle\Controller\IpsumApiController" public="true">
    <call method="setContainer">
        <argument type="service" id="Psr\Container\ContainerInterface"/>
    </call>
    <tag name="container.service_subscriber"/>
    <argument type="service" id="knpu_lorem_ipsum.knpu_ipsum"/>
</service>
```

For a full explanation, see this thread: https://bit.ly/abstract-controller-tag
***

Perfect!

## Routing

Finally, let's add some routing! In `Resources/config`, create a new `routes.xml`
file. This could be called anything because the user will import this file manually
from their app.

To fill this in, as usual, we'll cheat! Google for "Symfony Routing" and, just like
we did with services, search for "XML" until you find a good example.

Copy that code and paste it into our file. Let's call the one route
`knpu_lorem_ipsum_api`. For `controller`, copy the service id, paste, and add
a single colon then `index`.

[[[ code('8a905e7200') ]]]

Fun fact: in Symfony 4.1, the syntax changes to a double `::` and using a single
colon is deprecated. Keep a single `:` for now if you want your bundle to work in
Symfony 4.0.

Finally, for `path`, the user will probably want something like `/api/lorem-ipsum`.
But instead of *guessing* what they want, just set this to `/`, or at least, something
short. We'll allow the user to *choose* the path *prefix*.

And that's it! But... how can we make sure it works? In a few minutes, we're going
to write a *legitimate* functional test for this. But, for now, let's just test it
in our app!

In the `config` directory, we have a `routes.yaml` file, and we *could* import
the `routes.xml` file from here. But, it's more common to go into the `routes/`
*directory* and create a separate file: `knpu_lorem_ipsum.yaml`.

Add a root key - `_lorem_ipsum` - this is meaningless, then `resources` set to
`@KnpULoremIpsumBundle` and then the path to the file: `/Resources/config/routes.xml`.
*Then*, give this a prefix! How about `/api/ipsum`.

[[[ code('9f94feba69') ]]]

Did it work? Let's find out: find your terminal tab for the application, and use
the trusty old:

```terminal
php bin/console debug:router
```

There it is! `/api/ipsum/`. Copy that, find our browser, paste and.... nope.
Error!

> Controller ipsum_api_controller cannot be fetched from the container because it
> is private. Did you forget to tag the service with `controller.service_arguments`.

The error is not *entirely* correct for *our* circumstance. First, yes, at this time,
controllers are the *one* type of service that *must* be public. If you're building
an *app*, you can give it this tag, which will automatically make it public. But
for a reusable bundle, in `services.xml`, we need to set `public="true"`.

[[[ code('3d8899e75b') ]]]

Try that again! *Now* it works. And... you *might* be surprised! After all, our
bundle references a class that does *not* exist! This *is* a problem... at least,
a minor problem. But, because FrameworkBundle *is* included in our app, it *does*
work.

But to *really* make things solid, let's add a proper functional test to the bundle
that guarantees that this route and controller work. And when we do that, it'll
become *profoundly* obvious that we are, yet again, *not* properly requiring all
the dependencies we need.

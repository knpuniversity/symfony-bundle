# Adding Routes & Controllers

I *love* to talk about how the *whole* point of a bundle is that it adds *services*
to the container. But, really, a bundle can do a lot more than that: it can add
routes, controllers, transaction, public assets, validation config and a bunch more!

Find your browser and Google for "Symfony bundle best practices". This is a really
nice document that talks about how you're *supposed* to build a re-usable bundle.
I'm following, um, *most* of the recommendations. It tells you the different
directories where you should put different things. Some of these directories are
just convention, but some of them are required. For example, if your bundle provides
translations, they need to live in the `Resources/translations` directory next
to your bundle. If you follow that rule, they'll be automatically loaded.

## Adding a Route + Controller

Here's our next goal: I want to add a route & controller to our bundle. It'll
be an optional API endpoint that returns some lorem ipsum text. 

Before we start that, I'm going to open my PhpStorm preferences and, just to make
this more fun, search for "Symfony" and enable the Symfony plugin. Also search for
"Composer" and choose the `composer.json` file so that PhpStorm knows about our
autoload namespaces.

Back to work. In `src/`, create a `Controller` directory and inside of that, a
new PHP class called `IpsumApiController`. We don't need to make this extend anything,
but it's OK to extend `AbstractController` to get some shortcuts... except what!?
`AbstractController` doesn't exist!

That's because that class lives in `FrameworkBundle` and... remember! Our bundle
does *not* strictly require that! Ignore this problem for now. Instead, find out
app code, find `AbstractController`, copy its `namespace`, and use it to add the
`use` statement manually to our new controller.

Next, add a public function called `index`. Here, we're going to use the `KnpUIpsum`
class to return a JSON response with some dummy text. When you create a controller
in a reusable bundle, the best practice is to register your controller as a proper
service and use dependency injection to get anything you need.

Add `public function __construct()` and type-hint the first argument with `KnpUIpsum`.
I'll press Alt+Enter and choose initialize fields so that PhpStorm creates and
sets a property for that.

Down below, add return `$this->json()` - that won't auto-complete because of the
missing `AbstractController`, with a `paragraphs` set to
`$this->knpUIpsum->getParagraphs()` and a `sentences` key set to
`$this->knpUIpsum->getSentences()`

Excellent!

## Registering your Controller as a Service

Next, we need to register this as a service. In `services.xml`, copy the first
service, call this one `ipsum_api_controller`, and set its class name. For now,
*don't* add `public="true"` or `false`: I'll show you why in a minute. Pass the
service one argument: the main `knpu_lorem_ipsum.knpu_ipsum` service.

Perfect!

## Routing

Finally, let's add some routing! In `Resources/config`, create a new `routes.xml`
file. This could be called anything because the user is will import this file manually
from their app.

To fill this in, let's cheat as usual: Google for "Symfony Routing" and, just like
we did with services, search for "XML" until you find a good example.

Copy that file and paste it into our file. Let's call the one route
`knpu_lorem_ipsum_api`. For `controller`, copy the service id, paste, and add
a single colon then `index`.

Fun fact: in Symfony 4.1, the syntax changes to a double `::` and a single colon
is deprecated. Keep a single `:` for now if you want it to work in Symfony 4.0.

Finally, for `path`, the user will probably want something like `/api/lorem-ipsum`.
But, instead of guessing what they want, just set this to `/`, or at least, something
short. We'll allow the user to choose their path prefix.

And that's it! But... how can we make sure it works? In a few minutes, we're going
to write a *legitimate* functional test for this. But, for now, let's just try it
in our app!

In the `config` directory, we have a `routes.yaml` file, and we *could* import
the `routes.xml` file from here. But, its more common to go into the `routes/`
*directory* and create a separate file: `knpu_lorem_ipsum.yaml`.

Add a root key - `_lorem_ipsum` - this is meaningless, then `resources` set to
`@KnpULoremIpsumBundle` and the path to the file: `Resources/config/routes.xml`.
*Then*, give this a prefix! How about `/api/ipsum`.

Did it work? Let's find out: find your terminal tab for the application, and use
the trusty old:

```terminal
php bin/console debug:router
```

There it is! `/api/ipsum`. Let's copy that, find our browser, paste and.... nope.
Error!

> Controller ipsum_api_controller cannot be fetched from the container because it
> is private. Did you forget to tag the service with `controller.service_arguments`.

The error is not *entirely* correct for our circumstance. First, yes, at this time,
controllers are the *one* type of service I can think of that *must* be public.
If your building an app, you can give it this tag, which will automatically make
it public. For us, back on the bundle, in `services.xml`, we need to set `public="true"`.

Try that again! *Now* it works. And... you *might* be surprised! After all, our
bundle references a class that does *not* exist! This *is* a problem... at least,
a minor problem. But because FrameworkBundle *is* included in our app, it *does*
work.

But to *really* make things solid, let's add a proper functional test to the bundle
that guarantees that this route and controller work. And when we do that, it'll
become *profoundly* obvious that we are, yet again, *not* properly requiring all
the dependencies we need.

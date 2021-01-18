# Autowiring & Public/Private Services

Head back to `services.xml`: there are a *few* really important details we need to
get straight.

## Best-Practice Service IDs

First, in our *applications*, we usually make the service id match the class name
for simplicity: and that's what we've done here. But, when you create a re-usable
bundle, the best practice is to use *snake-case* service id's. Change the key to
`class` and add `id="knpu_lorem_ipsum.knpu_ipsum"`.

[[[ code('32c7db3eda') ]]]

*Why* is this the best practice? Well, the user *could* in theory change the class
of this service to one of their *own* classes. And, it would be pretty weird to have
a service called `KnpU\LoremIpsumBundle\KnpUIpsum`... when that's *not* actually
the class of the service.

## Supporting Autowiring

Anyways, this *simple* change, totally borks our app! Woohoo! Refresh!

It *once* again says that no service exists for `KnpUIpsum`. Remember: we're
*autowiring* that class into our controller. And in order for autowiring to work,
there *must* be a service whose id *matches* the class used in the type-hint. By
changing the id from the class to that weird, snake-case string, we just broke
autowiring!

No worries: we can solve this with a service *alias*. First, identify each service
in your app that you *intend* to be used directly by the user. Yea, I know, we only
have *one* service. But often, a bundle will have *several* services, but only *some*
of them are meant to be accessed by the user: the others are just meant to support
things internally.

For each "important" service, define an alias: `<service id="" ...>` and paste in
the class name. Then, `alias=""` and type the first service's id:
`knpu_lorem_ipsum.knpu_ipsum`.

[[[ code('b83abd4cbc') ]]]

To see what this did, move over to your terminal and run:

***TIP
In newer versions of Symfony, the `--show-private` option is not needed anymore!
***

```terminal
php bin/console debug:container --show-private knpu
```

Ok, there are *two* services: one has the snake-case id and the other is the full
class name. If you choose the second, *it's* just an *alias* to the snake-case
service. But now that there *is* a service whose id is the class name, anyone can
once again autowire using that type-hint. This fixes our page.

Yep, in `ArticleController`, the `KnpUIpsum` class is once-again autowired.

## Public versus Private Services

Ok, there is *one* last thing you need to think about when setting up your services:
whether or not each service should be *public* or *private*. In Symfony 4.0, services
are *private* by default, which means that a user cannot fetch a service directly
from the container with `$container->get()` and then the service's id. Instead, you
need to use dependency injection, which includes autowiring.

And this is *really* the way people should code going forward: we really should
*not* need services to be public. But, since some people *still* do fetch services
directly, you *may* want to make your important services public. Let's do this: `public="true"`.

[[[ code('c05d81bfe1') ]]]

And even though services are private by default, you should also add `public="false"`
to the others. This will make your services *also* behave the same on Symfony 3,
where they are *public* by default.

[[[ code('17c3537a96') ]]]

This makes no difference in our app - it all still works.

Alright! With our services configured, let's talk about how we can allow the user
to control the *behavior* of those services via configuration.

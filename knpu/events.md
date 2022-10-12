# Dispatching Custom Events

What if a user wants to change the behavior of our controller? Symfony *does* have
a way to override controllers from a bundle... but *not* if that controller is
registered as a service, like our controller. Well, ok, thanks to Symfony's incredible
container, there is *always* a way to override a service. But let's not make our
users do crazy things! If someone wants to tweak how our controller behaves, let's
make it easy!

How? By dispatching a custom event. Ready for our new goal? I want to allow a
user to *change* the data that we return from our API endpoint. Specifically,
we're going to add a *third* key to the JSON array from our app.

## Custom Event Class

The *first* step to dispatching an event is to create an event class. Create a
new `Event` directory with a PHP class inside: call it `FilterApiResponseEvent`.
I just made that up.

Make this extend a core `Event` class from Symfony. When you dispatch an event,
you have the opportunity to pass an Event object to any listeners. To be as
*awesome* as possible, you'll want to make sure that object contains as *much* useful
information as you can.

***TIP
Starting from Symfony 4.4, you should use the `Event` class from `Symfony\Contracts\EventDispatcher`:
If you want to know more about this: https://github.com/symfony/event-dispatcher/blob/4.4/Event.php
***


[[[ code('019ed33477') ]]]

In this case, a listener might want to access the data that we're about to turn 
into JSON. Cool! Add `public function __construct()` with an array `$data` argument.
I'll press Alt+Enter and choose "Initialize Fields" to create a data property and
set it.

[[[ code('c87c4a3be7') ]]]

Then, we need a way for the listeners to access this. *And*, we *also* want any
listeners to be able to *set* this. Go back to the Code -> Generate menu, or
Command + N on a Mac, choose "Getter and Setters" and select `data`.

[[[ code('b8401ee960') ]]]

It's ready!

## Dispatching the Event

Head to your controller: this is where we'll *dispatch* that event. First, set
the data to a `$data` variable and then create the event object:
`$event = new FilterApiResponseEvent()` passing it the data.

[[[ code('0e312e1f9b') ]]]

I'm not going to dispatch the event *quite* yet, but at the end, pass `$event->getData()`
to the `json` method.

[[[ code('1659a19cc7') ]]]

To dispatch the event, we need... um... the event dispatcher! And of course, we're
going to pass this in as an argument: `EventDispatcherInterface $eventDispatcher`.
Press Alt+enter and select "Initialize Fields" to add that as a property and set
it in the constructor.

[[[ code('522728d368') ]]]

As *soon* as we do this, we need to also open `services.xml` and pass a second
argument: `type="service"` and `id="event_dispatcher"`.

[[[ code('66dec817ab') ]]]

Back in the controller, right after you create the event, dispatch it:
`$this->eventDispatcher->dispatch()`. The first argument is the event *name* and
we can actually dream up whatever name we want. Let's use:
`knpu_lorem_ipsum.filter_api`. For the second argument, pass the event.

[[[ code('19be25e81c') ]]]

***TIP
Starting from Symfony 4.4, you should use instead:
```php
$this->eventDispatcher->dispatch($event, 'knpu_lorem_ipsum.filter_api');
```
***

And... yea, that's it! I mean, we haven't tested it yet, but this should work: our
users have a *new* hook point.

## Being Careful with Optional Dependencies

But actually there's a *small* surprise. Find your terminal and re-run all the tests:

```terminal-silent
./vendor/bin/simple-phpunit
```

They fail! Check this out: it says that our controller service has a dependency
on a non-existent service `event_dispatcher`. But, the service id *is*
`event_dispatcher` - that's not a typo! The problem is that the `event_dispatcher`
service - like *many* services - comes from `FrameworkBundle`.

Open up the test that's failing: `FunctionalTest`. Inside, we're testing with a
kernel that does *not* include FrameworkBundle! We did this on purpose: FrameworkBundle
is an *optional* dependency.

Let me say it a different way: one of our services depends on another service that
may or may not exist. Since we *want* our bundle to work without FrameworkBundle,
we need to make the `event_dispatcher` service optional. To do that, add an `on-invalid`
attribute set to `null`.

[[[ code('4203bdb5f7') ]]]

Thanks to this, if the `event_dispatcher` service doesn't exist, instead of an error,
it'll just pass `null`. That means, we need to make that argument optional, with
` = null`, or by adding a `?` before the type-hint.

[[[ code('470a8d6ffa') ]]]

Inside the action, be sure to code defensively: *if* there is an event dispatcher,
do our magic.

[[[ code('a9e7a71223') ]]]

Try the tests again:

```terminal-silent
./vendor/bin/simple-phpunit
```

Aw yea! Next, let's make our event *easier* to use by documenting it with an event
*constants* class. Then... let's make sure it works!

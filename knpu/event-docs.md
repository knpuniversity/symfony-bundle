# Event Constants & @Event Docs

There's one way we can make this better, and *all* high quality bundles do this:
set the event name as a *constant*, instead of just having this random string.
It's even a bit cooler than it sounds.

In the `Event` directory, create a new class: `KnpULoremIpsumEvents`. *If* your
bundle dispatches events, you should typically have *one* class that has a constant
for *each* event. It's a one-stop place to find *all* the event hook points.

Make this class `final`... which isn't too important... but in general, you should
considering making *any* class in a shareable library `final`, unless you *do* want
people to be able to sub-class it. Using `final` is always a safe bet and can be
removed later.

Anyways, add `const FILTER_API = ''`, go copy the event name and paste it here.

Now, of course, *replace* that string in the controller with
`KnpULoremIpsumEvents::FILTER_API`.

So, this is nice! Though, the reason I *really* like this is that it gives us a proper
place to document the *purpose* of this event: why you would listen to it and the
types of things you can do.

## The Special @Event Documentation

But the *coolest* part is this: add `@Event()`, and then inside double quotes,
put the full class name of the event that listeners will receive. In other words,
copy the namespace from the event class, paste it here and add `\FilterApiResponseEvent`.

What the heck does this do? On a technical level, absolutely nothing! This is purely
documentation. But! Some systems - like PhpStorm - know to *parse* this and use it
to help us when we're building event subscribers. We'll see *exactly* what I'm talking
about in a minute. But, it's at least good documentation: if you listen to this
event, this is the event object you should expect.

## Creating an EventSubscriber

And... we're done! I'm not going to write a test for this, but I *do* at least want
to make sure it works in my project. Move back over to the application code. Inside
`src/`, create a new directory called `EventSubscriber`. Then, a new class called
`AddMessageToIpsumApiSubscriber`.

Like all subscribers, this needs to implement `EventSubscriberInterface`. Then I'll
go to the Code -> Generate menu, or Command + N on a Mac, select Implement Methods,
and add `getSubscribedEvents`.

Before we fill this in, I want to make sure that PhpStorm is fully synchronized with
how our bundle looks - sometimes the symlink gets stale. Right click on the
`vendor/knpuniversity/lorem-ipsum-bundle` directory, and click "Synchronize".

Cool: now it will *definitely* see the new event classes. When it's done indexing,
return an array with `KnpULoremIpsumEvents::FILTER_API` set to, how about,
`onFilterApi`.

Ready for the magic? Thanks to the Symfony plugin, we can hover over the method name,
press Alt + Enter and select "Create Method". Woh! It added the `onFilterApi` method
for me *and* type-hinted the first argument with `FilterApiResponseEvent`! But, how
did it know that this was the right event class?

It knew that thanks to the `@Event()` documentation we added earlier.

Inside the method, let's say `$data = $event->getData()` and then add a new key
called `message` set to, the very important, "Have a magical day". Finally, set
that data *back* on the event with `$event->setData($data)`.

That is it! Thanks to Symfony's service auto-configuration, this is already a
service and it will already be an event subscriber. In other words, go refresh the
API endpoint. It, just, works! Our controller is now extensible, without the user
needing to override it. Dispatching events is most commonly done in controllers,
but you could dispatch them in any service.

Next, let's improve our word provider setup by making it a true *plugin* system
with dependency injection tags and compiler passes. Woh.

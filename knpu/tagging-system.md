# Plugin System with Tags

At this point, the user *can* control the word provider. But, there's only ever
*one* word provider. That may be fine, but I want to make this more flexible! And,
along the way, learn about one of the most important, but complex systems that is
commonly used in bundles: the tag & compiler pass system.

First, let's make our mission clear: instead of allowing just *one* word provider,
I want to allow *many* word providers. I also want *other* bundles to be able to
automatically add new word providers to the system. Basically, I want a word provider
*plugin* system.

## Allowing Multiple Word Providers

To get this started, we need to refactor `KnpUIpsum`: change the first argument
to be an *array* of `$wordProviders`. Rename the property to `$wordProviders`,
and I'll add some PHPDoc above this to help with auto-completion: this will be
an array of `WordProviderInterface[]`.

Let's also add a *new* property called `wordList`: in a moment, we'll use this to
store the final word list, so that we only need to calculate it once.

The big change is down below in the `getWordList()` method. First, if
`null === $this->wordList`, then we need to loop over all the word providers to
*create* that word list.

Once we've done, that, at the bottom, return `$this->wordList`.

Back in the if, create an empty `$words` array, then loop over `$this->wordProviders`
as `$wordProvider`. For each word provider, set `$words` to an `array_merge` of
the words so far and `$wordProvider->getWordList()`.

After, we need a sanity check: if the `count($words) <= 1`, throw an exception:
this class only works when there are at least *two* words. Finally, set
`$this->wordList` to `$words`.

Perfect! This class is now just a *little* bit more flexible. In `config/services.xml`,
instead of passing one word provider, add an `<argument` with `type="collection"`,
them move the word provider argument inside of this.

There's no fancy plugin system yet, but things *should* still work. Find your browser
and refresh. Great! Even the article page looks fine.

## Tagging the Service

Here's the burning question: how can we improve this system so that our application,
or even *other* bundles, can add new word providers to this collection? The answer...
takes a few steps to explain.

First, I want you to pass an *empty* collection as the first argument. Then, below
on the word provider service, change this to use the longer service syntax so that,
inside, we can add `<tag name="">`, and, invent a new tag string. How about:
`knpu_ipsum_word_provider`.

If this makes *no* sense to you, no problem. Because, it will *not* work yet: when
you refresh, big error! At this moment, there are *zero* word providers.

If you've worked with Symfony for awhile, you've probably *used* tags before. At
a high-level, the idea is pretty simple. First, you can attach tags to services...
which... initially... does nothing. But then, a bundle author - that's us! - can
write some code that finds all services in the container with this tag and dynamically
add them to the collection argument!

When this is setup, our application - or even *other bundles* - can add services,
give them this tag, and they will automatically be "plugged" into the system.
This is how Twig Extensions, Event Subscribers, Voters, and many other parts of
Symfony work.

## The Easy Way

So... how do we hook this all up? Well, if your bundle will only need to support
Symfony 3.4 or higher, there's a *super* easy way. Just replace the
`<argument type="collection">` with
`<argument type="tagged" tag="knpu_ipsum_word_provider" />`. This tells Symfony
to find all services with this tag, and pass them as a collection. And... you'd
be done!

But, if you want to support *earlier* versions of Symfony, or you want to know
how the compiler pass system works, keep watching.

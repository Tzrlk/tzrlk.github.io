---
layout: post
title: Understandable Frustration
tags:
  - testing
  - rant
date: 2016-11-07T00:00:00.000Z
thumbnail: null
published: true
---

With any luck, venting about this here will lead me to the correct answer via the highly effective process of [Rubber Duck Debugging](https://en.wikipedia.org/wiki/Rubber_duck_debugging).

So, I’ve been attempting to convert the haphazard pile of bash scripts used to prevent bad pushes to the git repository at work into an invoked cli jar compiled from kotlin. For the most part, the conversion process has gone well. I haven’t had the chance to properly refactor and reengineer it yet, but that should come after I can get the damn tests passing.

I had already been using [shunit2](http://lmgtfy.com/?q=shunit2) via [my own 'fork' of it](https://github.com/aetheric/shunit2) to make sure all the bash scripts worked like I expected, but as you might imagine, that was super clunky to run and maintain. Converting everything to kotlin was fairly straightforward, and I ended up using a neat set of libraries:

-   [hamkrest](https://github.com/npryce/hamkrest)

-   [springockito](https://github.com/springockito/springockito)

-   [mockito-kotlin](https://github.com/nhaarman/mockito-kotlin)

-   [junit-quickcheck](https://github.com/pholser/junit-quickcheck)

-   [junit-nested](https://github.com/avh4/junit-nested)

Haven’t ended up getting too much use out of quickcheck just yet, but mockito-kotlin and hamkrest have been amazing in a kotlin environment. They’re little more than a thin veneer on top of the relevant libraries, but it makes things much simpler to write.

The thing that’s driving me nuts at the moment, however, is a pair of tests keep failing for a clear reason, but without the needed context to fix it. I even ended up creating a series of wrapper classes to avoid needing to mock things like RevCommit and ObjectId (jgit), but to no avail. Each and every time I run the damn thing I get the same set of errors.

The problem is the stacktrace ends up being useless, since it’s only points to the place where the library raised the error, rather than the place that the error is complaining about. A few libraries like angularjs are notorious for this complete lack of context in their errors, meaning you end up waiting for the error to be thrown, then debugging backward through the process until you find what’s wrong.

Of course, in my case, the problem [is like an onion](https://www.youtube.com/watch?v=BUsSPAFmauY&feature=youtu.be&t=33s). Every time I think I’ve stumbled across what’s causing the damn errors, I just uncover a new one that makes just as little sense as the first. Observe:

    java.lang.ClassCastException:
        org.mockito.internal.creation.cglib.ClassImposterizer
        $ClassWithSuperclassToWorkAroundCglibBug
        $EnhancerByMockitoWithCGLIB
        $$8b138c7f cannot be cast to
        ...tools.git.wrappers.ObjectIdWrapper

I love how descriptive the class names are, but it tells me almost nothing about why the hell this is happening. Casting that class should be perfectly fine if its 'extending' ObjectIdWrapper. The only thing I can think of is that it’s a bug somewhere between mockito and kotlin, and if that’s the case, then I should just flip a table and give up trying to have testable code.

Of course, in my efforts to get one test working, I end up breaking a bunch of other ones. Fml. Maybe I’ll have a flash of inspiration, but seeing how things have progressed so far makes that a very dim possibility.

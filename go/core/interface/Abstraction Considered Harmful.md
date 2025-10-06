https://bravenewgeek.com/abstraction-considered-harmful/

_“Abstraction is sometimes harmful,”_ he proclaims to the sound of anxious whooping and subdued applause from the audience. Peter Alvaro’s 2015 Strange Loop keynote, _[I See What You Mean](https://youtu.be/R2Aa4PivG0g)_, remains one of my favorite talks—not just because of its keen insight on distributed computing and language design, but because of a more fundamental, almost _primordial_, understanding of systems thinking.

Abstraction is what we use to manage complexity. We build something of significant complexity, we mask its inner workings, and we expose what we think is necessary for interacting with it.

Programmers are lazy, and abstractions help us _be_ lazy. The builders of abstractions need not think about how their abstractions will be used—this would be _far_ too much effort. Likewise, the users of abstractions need not think about how their abstractions work—this would be _far_ too much effort. And now we have a nice, neatly wrapped package we can use and reuse to build all kinds of applications—after all, duplicating it would be _far_ too much effort and goes against everything we consider sacred as programmers.

It usually works like this: in order to solve a problem, a programmer first needs to solve a sub-problem. This sub-problem is significant enough in complexity or occurs frequently enough in practice that the programmer doesn’t want to solve it for the specific case—_an abstraction is born_. Now, this can go one of two ways. Either the abstraction is rock solid and the programmer never has to think about the mundane details again (think writing loops instead of writing a bunch of _jmp_ statements)—_success_—or the abstraction is leaky because the underlying problem is sufficiently complex (think distributed database transactions). _Infinite sadness_.

It’s kind of a cruel irony. The programmer complains that there’s not enough abstraction for a hard sub-problem. Indeed, the programmer doesn’t care about solving the sub-problem. They are focused on solving the greater problem at hand. So, as any good programmer would do, we build an abstraction for the hard sub-problem, mask its inner workings, and expose what we think is necessary for interacting with it. But then we discover that the abstraction _leaks_ and complain that it isn’t perfect. It turns out, hard problems are _hard_. The programmer then simply does away with the abstraction and solves the sub-problem for their specific case, handling the complexity in a way that makes sense for their application.

Abstraction doesn’t magically make things less hard. It just attempts to hide that fact from you. Just because the _semantics_ are simple doesn’t mean the _solution_ is. In fact, it’s often the opposite, yet this seems to be a frequently implied assumption.

[Duplication is far cheaper than the wrong abstraction](http://www.sandimetz.com/blog/2016/1/20/the-wrong-abstraction). Just deciding which little details we need to expose on our abstractions can be difficult, particularly when we don’t know _how_ they will be used. The truth is, we _can’t_ know how they will be used because some of the uses haven’t even been _conceived_ yet. Abstraction is a delicate balance of precision and granularity. To [quote Dijkstra](https://www.cs.utexas.edu/~EWD/transcriptions/EWD03xx/EWD340.html):

> The purpose of abstracting is not to be vague, but to create a new semantic level in which one can be absolutely precise.

But, as we know, requirements are fluid. Too precise and we lose granularity, hindering our ability to adapt in the future. Too granular and we weaken the abstraction. But a strong abstraction for a hard problem isn’t really strong at all when it leaks.

The key takeaway is that abstractions leak, and we have to deal with that. There is never a silver bullet for problems of sufficient complexity. Peter ends his talk on a polemic against the way we currently view abstraction:

> _[Let’s]_ not make concrete, static abstractions. Trust ourselves to let ourselves peer below the facade. There’s a lot of complexity down there, but we need to engage with that complexity. We need tools that help us engage with the complexity, not a fire blanket. Abstractions are going to leak, so make the abstractions fluid.

Abstraction, in and of itself, is not harmful. On the contrary, it’s _necessary_ for progress. What’s harmful is relying on _impenetrable barriers_ to protect our precious programmers from hard problems. After all, the [21st century engineer](https://bravenewgeek.com/infrastructure-engineering-in-the-21st-century/) understands that in order to play in the sand, we all need to be comfortable getting our feet a little wet from time to time.
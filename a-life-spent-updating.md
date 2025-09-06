# [A life spent updating](https://www.city17.xyz/a-life-spent-updating/)

While I was doing the routine weekend maintenance of my electronic devices, I paused and thought for a second about what I was doing.

How often do I update my stuff? How much stuff do I update regularly?

1.  A laptop:

- Debian/testing system updates
- Firefox updates
- Firefox extensions updates
- Rust compiler updates

2.  A desktop:

- Debian/testing system updates
- Rust compiler updates

3.  A smartphone:

- OS updates
- Google Play Apps updates
- F-droid Apps updates

4.  A server:

- Debian/stable updates (including some user-facing services, like Postgres or nginx)
- A number of different Docker containers with different release cycles

It's a lot of chore! I probably notice all this because I don't do automatic updates and I always manually review them, possibly even reading the changelog in case of critical packages.

Long time ago, once software was published it was finished, for the good or the worse. Users wanting new features of bugfixes had to wait months (if not years!) for an update or a new release. Today, software is never finished, it is a never-ending burden on users. And while I am among the lucky ones (compared to those having more devices/workstations to attend to, not to mention users on rolling release Linux distributions!), I can only frown upon on the amount of work requested to us. What did we give up in exchange of a quicker development/release pace?

Much like a long bridge built on toothpicks, dozen of thousands of unaware commuters drive and walk daily on this bridge without realizing the inherent complexity and its fragility. Only when the occasional software disaster hits the mainstream media, and many people get burned, these commuters may get a glimpse of this complexity. The reaction, though, is to accept as a fact of life that things break and someone, somewhere, will fix the problem so we can all move on. Today's systems are complex (just look at a smartphone!) but I think it's not a good reason to avoid asking questions about what we can do to improve the reliability and reduce the complexity of these systems.

Concretely, I would like:

- **Software to reach a state of finished product**: this is unfortunately incompatible with the development model that is often open and incremental. Though nothing blocks open-source authors to stop adding features and once satisfied just do maintenance. Likewise, users should learn to not expect authors to furiously add code upon code to avoid perceiving a project as "abandoned".
- **Define a scope for software**: corollary of the previous point. Authors should prefer defining a scope rather than haphazardly developing and endless stream of code. Also here there is a trap: users might prefer a half-assed package aiming to be the _all-encompassing-app-the-one-and-only-you-will-ever-need_ than a finished but feature-limited application.
- **Avoid adding useless stuff**: another corollary of the first point. A bit more nuanced because what for me is _useless_, for someone else can be a nice-to-have. In general, I believe software should be well-thought and should not run behind the latest trends[^1].
- **Dilute release cycles**: Software updates, unless critical, can also happen at a more relaxed spacing. Users learn that as soon as a fix/feature is ready (for various meaning of "ready"), the authors will push it outside. These are expectations that we should unlearn.

More in general, I wish software to be _updated_ only for fixes and reasonable small improvements and not push on users updates that in reality are unrequested _upgrades_ to new versions, which sometimes brings new bugs[^2]. I want to be in charge of whether to install a version with new features. Consumer software is especially the culprit here because users have lower expectations, often don't pay any money so we are treated as second-class citizens. Not to mention the subject of SaaS, software hosted on the cloud, where users have absolutely no control on what versions they are using.

## [§](https://www.city17.xyz/a-life-spent-updating/#conclusions) Conclusions

I am under no illusion that the current trend is not going to change anytime soon and I am pessimistic about software being a furnace that needs to be continuously fed with coal, giving something to do to a lot of people (like me).

Currently I can only try to keep under control this problem by carefully choosing my software providers, endorsing those that prioritize (or at least balance) stability over releasing new stuff. In this regard I am quite happy with the Debian/testing release channel. Unfortunately I can't say the same of what happens on my smartphone.

As I sometimes mention in my blog, Sourcehut takes seriously adding features and I wish more projects could learn a thing or two from Sourcehut. Something that works just does not _stop working_ for any reason. I agree that Sourcehut is also so stable because there are currently very few resources[^3] to push it out of its alpha state, so in the end we have a very good MVP that doesn't improve nor break.

[^1]: As an example, Protonmail in 2022 [held a poll](https://proton.me/blog/2022-proton-survey-insights) asking users what direction should they prioritize development. Users expressed their opinions and thing moved presumably in that direction. But today we have an [AI assistant](https://proton.me/blog/proton-scribe-writing-assistant) suggesting how to write an email even though nobody asked for it (we were not yet in 2024, the AI craze!). [↩](https://www.city17.xyz/a-life-spent-updating/#fr-1-1)
[^2]: [https://github.com/cinnyapp/cinny/issues/1844](https://github.com/cinnyapp/cinny/issues/1844) [↩](https://www.city17.xyz/a-life-spent-updating/#fr-2-1) 
[^3]: [https://sourcehut.org/blog/2024-06-04-status-and-plans](https://sourcehut.org/blog/2024-06-04-status-and-plans) [↩](https://www.city17.xyz/a-life-spent-updating/#fr-3-1)

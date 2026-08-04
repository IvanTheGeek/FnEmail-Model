# Event Modeling — Source Bibliography

Compiled from a research sweep whose network access was limited to `github.com` and `raw.githubusercontent.com` plus web search. Entries drawn from the eventmodeling.org Hugo source repository were read verbatim from raw Markdown and are high confidence. Entries on blocked hosts (LinkedIn, InfoQ, ACM, Leanpub, Semaphore, podcast pages, YouTube, conference sites) were established through search results only and are flagged where confidence is lower.

**Attribution warning, up front:** only three published pages on eventmodeling.org are Adam Dymitruk's own writing — the home page, "Event Modeling: What is it?", and "Event Modeling Traditional Systems". The widely-cited *Event Modeling Cheat Sheet* — origin of the now-standard "four building blocks / four named patterns" formulation — is by **Sebastian Bortz**, not Dymitruk. This is confirmed from the site's own `data/authors.toml`.

---

## Adam Dymitruk

### Canonical site & articles

- [Event Modeling: What is it?](https://eventmodeling.org/posts/what-is-event-modeling/) — 2019 (front matter 2019-06-23; periodically updated, last content commit 2025-10-24), eventmodeling.org. The foundational and by far the most substantive Event Modeling text: first-principles argument (cheap storage, human memory works by story), the building blocks, the four patterns, the seven-step workshop format, and the economic case (flat cost curve, estimates-without-estimating, fixed-cost subcontracting). This is the only source for the canonical seven steps, under the heading "Workshop Format - The 7 Steps" (anchor `#seven-steps`), and for the connection rule: "command to event, event to state view, state view to UI or processor, UI/processor to command. Connecting an event directly to a command is not allowed." Note that it counts **three** building blocks, not four. Ends "(to be continued)".

- [Event Modeling Introduction (site home page)](https://eventmodeling.org/) — 2020 (front matter 2020-04-22), eventmodeling.org. The one-sentence definition everyone quotes: "Event Modeling is a method of describing systems using an example of how information has changed within them over time. It was developed at Adaptech Group by Adam Dymitruk." Also states the core economic problem (rising cost curve driven by rework) and that the method is not tied to event sourcing.

- [About — Event Modeling](https://eventmodeling.org/about/) — 2019 (front matter 2019-06-23), eventmodeling.org. The primary source for the method's lineage: coined by Dymitruk "by building on long-running process specifications that Greg Young used in CQRS/ES systems", with Alberto Brandolini's Event Storming supplying the workshop format and sticky notes, plus the UI/UX storyboard layer. Fixes the naming: "The name 'Event Modeling' was established after the Event Storming Summit in July 2018 in Bologna Italy." Also carries the canonical distinction — Event Storming discovers the problem space, Event Modeling produces a solution blueprint.

- [Event Modeling Traditional Systems](https://eventmodeling.org/posts/event-modeling-traditional-systems/) — 2020 (front matter 2020-03-14), eventmodeling.org. The companion piece: applies the method to systems persisting state in ordinary relational tables. Opens on Fred Brooks ("Show me your flowcharts and conceal your tables…"), coins **Small Design Up Front (SDUF)** against BDUF, and makes the rework cascade concrete with a lettered feature-dependency example.

- [Specifying Complex Views](https://raw.githubusercontent.com/event-modeling/eventmodeling.org/master/content/posts/specifying-complex-views/index.md) — 2020 (front matter 2020-05-21), eventmodeling.org repo. **Unpublished draft** (`draft: true`), not rendered on the live site. A five-line new introduction about specifying view permutations sits atop a byte-identical copy of "Event Modeling Traditional Systems". Recorded so it is not mistaken for a distinct published article.

- [Resources — Event Modeling](https://eventmodeling.org/resources/) — front matter 2019-06-27, content updated through at least Nov 2024, eventmodeling.org. Dymitruk's own curated index of the ecosystem: endorses Martin Dilger's book as the learning path, points to the podcast and the Nebulit newsletter, lists the Discord (`https://discord.gg/Sw4MvagftJ`), the printable laptop-sticker SVGs, the Miro template, and the tool list (Nebulit Miro plugin, ONote, Evident Design, Modellution). Also embeds the talks he treats as canonical introductions.

- [Event Modeling: Designing Modern Information systems (LinkedIn Pulse)](https://www.linkedin.com/pulse/event-modeling-designing-modern-information-systems-adam-dymitruk) — year unverified; the LinkedIn `articleId=6551324696745512960` falls in a 2019 range. Under Dymitruk's byline; the title matches the site's subtitle, so this is almost certainly a republication of "Event Modeling: What is it?" rather than distinct material. Body text not verified — LinkedIn was unreachable.

- [Event Modeling Traditional Systems (LinkedIn Pulse)](https://www.linkedin.com/pulse/event-modeling-traditional-systems-adam-dymitruk) — year unverified; the eventmodeling.org twin is dated 2020-03-14. LinkedIn republication under Dymitruk's byline; slug matches the site post exactly. Independently cited at [MateuszNaKodach/SelfImprovement#3576](https://github.com/MateuszNaKodach/SelfImprovement/issues/3576). Body text not verified.

- [Adaptech Group](https://adaptechgroup.com/) — current. Dymitruk's consultancy and the organization eventmodeling.org names as where the method "was developed"; workshop booking lives at the `#workshops` anchor. The site could not be fetched, so any whitepapers or written material it hosts remain uncatalogued.

- [dymitruk.com — personal blog](http://dymitruk.com/) — 2012 era. Contains **no Event Modeling content**. Parsing `atom.xml` from its GitHub repo shows exactly five posts, all 2012, chiefly the influential [Branch Per Feature](http://www.dymitruk.com/blog/2012/02/05/branch-per-feature/) git strategy piece. Listed so a reference document can affirmatively rule it out as an EM source.

### Talks & podcasts

- [SE Radio 539: Adam Dymitruk on Event Modeling](https://se-radio.net/2022/11/episode-539-adam-dymitruk-on-event-modeling/) — November 2022, Software Engineering Radio (IEEE Computer Society), host Jeff Doolittle. Dymitruk walks through Event Modeling as an approach to requirements discovery and system design, framed around removing implementation and technology decisions from the design conversation. The episode page carries a machine-generated transcript; also on [YouTube](https://www.youtube.com/watch?v=x3o7g3PJgfY) and written up by the host at [jeffdoolittle.com](https://jeffdoolittle.com/2022/11/22/event-modeling-with-adam-dymitruk/).

- [Exploring Event Modeling with Adam Dymitruk (Hanselminutes #771)](https://hanselminutes.com/771/exploring-event-modeling-with-adam-dymitruk) — 14 January 2021, host Scott Hanselman. Contains the clearest spoken statement of the method's lineage — that Dymitruk built it on the long-running-process specifications Greg Young used in CQRS/ES systems. Also distributed via [Simplecast](https://hanselminutes.simplecast.com/episodes/exploring-event-modeling-with-adam-dymitruk-xvAdQlCd), [Apple Podcasts](https://podcasts.apple.com/au/podcast/exploring-event-modeling-with-adam-dymitruk/id117488860?i=1000505378550) and [dev.to](https://dev.to/hanselminutes/exploring-event-modeling-with-adam-dymitruk).

- [Exploring Axon Episode 8: Event Modeling](https://exploringaxon.podbean.com/e/event-modeling/) — 30 October 2020, AxonIQ. The most structurally detailed early podcast: segment list covers the full process with examples, Event Storming vs Event Modeling, Input/Output, States, Limitations, Given-When-Then, State Change and State View, Translation Patterns, Automation, Cost, Agile, and Blueprint. Also at [axoniq.io](https://www.axoniq.io/podcasts/event-modeling) and [YouTube](https://www.youtube.com/watch?v=BZk35QFjYmg).

- [Event Modeling • Adam Dymitruk • YOW! 2022](https://www.youtube.com/watch?v=cVyVmcwiPWw) — 2022, YOW! Conference Australia, published on the GOTO Conferences channel. Covers how the method works, how to organize effective workshops, and application across project sizes; the [Class Central index entry](https://www.classcentral.com/course/youtube-event-modeling-adam-dymitruk-yow-2022-193486) lists blueprints, alignment, feedback, planning meetings, change management, and the relationship to DDD and event-driven architecture.

- [Hands-On Event Modeling (masterclass) — YOW! Sydney 2023](https://yowcon.com/sydney-2023/masterclasses/432/hands-on-event-modeling) — 2023, co-delivered with Alexandra Moxin. Full-day paid masterclass: design a system in the morning, implement it in the afternoon, using pen-and-paper and online collaborative tooling, with product owners adjusting the model live during the build phase. Almost certainly not recorded.

- [Event Modeling Conference 2025 (Munich) — opening keynote](https://nebulit.de/eventmodeling-conference-2025) — October 2025, Impact Hub Munich, organized by Nebulit / Martin Dilger. Keynote (09:15–10:15) on the principles, origins and philosophy of the method, delivered with no slides at all per the [organizer's recap](https://www.eventmodelers.ai/docs/blog/event-modeling-conference-munich/); followed by a hands-on workshop co-led with Dilger. Announced a forthcoming Event Modeling certification program and a joint venture between Dymitruk and Dilger focused on tooling and standardization.

- [Event Modeling Conference 2026 (Munich) — keynote](https://www.nebulit.de/eventmodeling-conference-2026) — scheduled Thursday 25 June 2026, 09:15–10:15. Listed as keynote speaker; whether the session took place as scheduled and whether any recording was published could not be confirmed.

- [Event Modeling: The Blueprint for Building Maintainable Software — with Adam Dymitruk](https://newsletter.nerdnoir.com/p/event-modeling-the-blueprint-for) — 14 April 2026, Nerd/Noir. The most recent substantive interview found. Contains the clearest statement of the blueprint analogy (plumber, carpenter and electrician all working from one document) and a book status update: Dymitruk is writing the definitive book on Event Modeling with a companion booklet due out first.

- [The Event Modeling and Event Sourcing Podcast](https://podcast.eventmodeling.org/) — 2024–2026, ongoing, co-hosted with Martin Dilger. Weekly, launched 4 November 2024 with [Ep 1 "Destroying the Aggregate"](https://podcast.eventmodeling.org/episodes/episode-1/); 43+ episodes as of the sweep. The single largest body of recorded material from either author. Notable episodes: [Ep 19 "Vibe Modeling, Event Models for the C-Suite"](https://podcast.eventmodeling.org/episodes/episode-19/), [Ep 25 "Conceptual Structure of Code"](https://podcast.eventmodeling.org/episodes/episode-25/), [Ep 26 "A New Programming Language"](https://podcast.eventmodeling.org/episodes/episode-26/), [Ep 32 "2026 is the Year of the Event Modeling Desktop!"](https://podcast.eventmodeling.org/episodes/episode-32/), [Ep 39 "Event Sourcing Predates Anything in Computing"](https://podcast.eventmodeling.org/episodes/episode-39/), [Ep 41 "Reverse Engineering Using Event Modeling"](https://podcast.eventmodeling.org/episodes/episode-41/). Feeds: [RSS](https://podcast.eventmodeling.org/index.xml), [YouTube](https://www.youtube.com/@eventdrivenpodcast), [Spotify](https://open.spotify.com/show/5yIPErSiVqs2fkMia3WbwZ), [Podcast Addict](https://podcastaddict.com/podcast/5822772).

- [Interview with Event Modeling Founder - Adam Dymitruk (InfoQ)](https://www.infoq.com/news/2020/09/adameventmodeling/) — September 2020, InfoQ. A written Q&A, so the answers are Dymitruk's own prose: Event Modeling "was born out of the need to make software development — especially in the enterprise — to be more like an engineering discipline", and "the industry gave up on design in lieu of agile when the case was made against onerous RUP and UML". A [Japanese translation](https://www.infoq.com/jp/news/2020/10/adameventmodeling/) exists (October 2020). Quotes come from search summaries — InfoQ was unreachable — and should be spot-checked before verbatim reuse.

- [Adam Dymitruk on How to Upgrade Your Toolbox with Event Modeling (Semaphore Uncut)](https://semaphore.io/blog/adam-dymitruk-event-modeling) — 2022. Transcribed podcast interview; spoken-then-transcribed rather than authored prose, but contains quotable positions on event sourcing, DDD ("use subject matter experts in each area properly so that the terminology keeps consistent") and vertical slicing. Republished on [Medium](https://semaphoreci.medium.com/adam-dymitruk-on-how-to-upgrade-your-toolbox-with-event-modeling-c131d27c712b) and [dev.to](https://dev.to/semaphoreuncut/adam-dymitruk-on-how-to-upgrade-your-toolbox-with-event-modeling). Quotes are from search summaries, unverified against the page.

- [Adam Dymitruk — Event Modelling, Event Sourcing, CQRS | The Technologist Podcast #8](https://m.youtube.com/watch?v=i8RwL0bLJAU) — approx. October 2022. Covers Event Modelling, Event Sourcing and CQRS. The episode framing places Dymitruk among the pioneers of Event Sourcing and CQRS alongside Greg Young and Martin Fowler — this is the host's characterization, not necessarily Dymitruk's own claim.

- [Event Modeling with Adam Dymirtuk (.NET Rocks! show 1928)](https://www.dotnetrocks.com/default.aspx?ShowNum=1928) — year unverified, likely 2024/2025 (note the show title misspells the surname). Framed around entering event sourcing through business workflows rather than storage technology. Also on [Spotify](https://open.spotify.com/episode/6v22Nq154fWpRJDCzPdW4n) and [Spreaker](https://www.spreaker.com/episode/event-modeling-with-adam-dymirtuk--63278642). The show number was resolved via search, not by reading the episode page.

- [The Loosely Coupled Show: Event Modeling with Adam Dymitruk](https://creators.spotify.com/pod/profile/loosely-coupled-show/episodes/Event-Modeling-with-Adam-Dymitruk-elo7m3) — date not established. Software architecture and design show hosted by James Hickey and Derek Comartin.

- [Adam Dymitruk — DDD TW Conference 2021 speaker page](https://conference.ddd-tw.com/2021/speakers/adymitruk/) — 2021, Domain-Driven Design Taiwan. Speaker page; a search summary reported the talk title as "Event Modeling as a Way to Entirely Manage the Software Development Life-Cycle" but the page could not be opened to confirm it, and no recording was found.

- [Vancouver Tech Podcast Ep. 70: Adam Dymitruk of Adaptech Solutions](https://podcasts.apple.com/ca/podcast/episode-70-adam-dymitruk-of-adaptech-solutions/id1057679825?i=1000384180303) — 2017, host Drew Ogryzek. Predates the naming of Event Modeling: covers Event Sourcing, pattern-following in software problem solving, building a consulting business, and responsibly onboarding junior developers. Covered by [BetaKit](https://betakit.com/vancouver-tech-podcast-ep-70-adam-dymitruk-ceo-of-adaptech-solutions/).

- [Vancouver Tech Podcast Ep. 78: Adam Dymitruk on Monoliths vs. Microservices](https://betakit.com/vancouver-tech-podcast-ep-78-adaptech-solutions-adam-dymitruk-on-monoliths-vs-microservices/) — 30 May 2017. What causes monoliths, how to avoid them, and a contrarian take treating technical debt as an investment rather than a debt when it is contributing business value. Predates Event Modeling but establishes positions carried forward.

- [Adam Dymitruk on Event Modeling (Communications of the ACM)](https://cacmb4.acm.org/opinion/interviews/267102-adam-dymitruk-on-event-modeling/fulltext) — year unverified, likely 2022. *(unverified)* Interview covering the structured approach to requirements discovery. The host `cacmb4.acm.org` looks like an ACM staging or backup domain rather than canonical `cacm.acm.org`; no canonical equivalent could be surfaced and neither could be fetched. Expect this URL to need re-resolution.

### Repositories

- [event-modeling/eventmodeling.org](https://github.com/event-modeling/eventmodeling.org) — created 2019-06-23, content last touched 2025-10-24. The complete Hugo source of eventmodeling.org in Markdown, publicly readable and fully mirrorable — the basis for every verbatim quote in this bibliography. Contains exactly six posts in source, five published (`specifying-complex-views` is `draft: true`), plus `data/authors.toml`, which is the authoritative record of who wrote what. Recent commits ("initial wording changes. needs further updates and diagram updates", Oct 2025) suggest a revision of the canonical article is in progress but unlanded.

- [adymitruk/adymitruk.github.com](https://github.com/adymitruk/adymitruk.github.com) — 2012 era. Octopress source for dymitruk.com (CNAME = dymitruk.com). Fully mirrorable; contains no Event Modeling material.

### Tools

- [Implementation gist — fish-shell Trello example](https://gist.github.com/adymitruk/7fc2adb8598ad861d4b3dae114afd4c9) — Dymitruk's own worked implementation example, linked from the Resources page. URL confirmed reachable (HTTP 200).

---

## Martin Dilger

### Books

- [Understanding Eventsourcing: Planning and Implementing scalable Systems with Eventmodeling and Eventsourcing](https://leanpub.com/eventmodeling-and-eventsourcing) — 2024, first published, continuously updated, Leanpub. Dilger's primary work and the book eventmodeling.org officially endorses as the way to learn the practice. Four parts: distributed-systems/CQRS/event-sourcing foundations; modelling with Event Modeling; implementation in code; and a Pattern Catalog of recipes for typical event-sourced scenarios. Its explicit extension over Dymitruk is the vertical-slice framing — it claims to be "the first book to combine Eventmodeling, Eventsourcing and Vertical Slice Architectures in one consistent software development process", treating Event Modeling as the planning phase and Event Sourcing as the implementation phase of a single method. The Leanpub edition is a living book with chapters added roughly every 14 days, so it diverges from and exceeds the print edition; later chapters cover metadata, security, GDPR, UI for event-sourced systems, the Decider pattern, API design and legacy systems. Leanpub was unreachable; contents established from search.

### Talks & podcasts

- [The Event Modeling and Event Sourcing Podcast](https://podcast.eventmodeling.org/) — 2024–2026, co-hosted with Adam Dymitruk. See the full entry under Adam Dymitruk → Talks & podcasts.

- [Event Modeling Conference 2025 (Munich)](https://nebulit.de/eventmodeling-conference-2025) — October 2025. Dilger, through Nebulit, organized the first dedicated Event Modeling conference (~40 practitioners, sold out) and co-led the hands-on workshop with Dymitruk. The [conference recap](https://www.eventmodelers.ai/docs/blog/event-modeling-conference-munich/) is the source for the certification-program and joint-venture announcements.

- [Event Modeling Conference 2026 (Munich)](https://www.nebulit.de/eventmodeling-conference-2026) — June 2026, organized by Nebulit.

---

## Other / Community

### Canonical site & articles (guest-authored on eventmodeling.org)

- [Event Modeling Cheat Sheet](https://eventmodeling.org/posts/event-modeling-cheatsheet/) — 2020-12-09, marked "Updated article on 08-05-2022", by **Sebastian Bortz** (front-matter author `sbortz`). This is the piece everyone means by "the Event Modeling cheat sheet", and it is the origin of the now-universal formulation: **four building blocks** (Trigger, Command, Event, View) and **four named patterns** (Command Pattern = Trigger→Command→Event(s); View Pattern = Event(s)→View; Automation Pattern = Event(s)→View→Automated Trigger→Command→Event(s); Translation Pattern = the same shape across system boundaries). Also fixes the colour convention (white Trigger, blue Command, yellow Event, green View) and the definition of a slice. Careful about attribution itself — it reports what "Adam teaches" — but downstream writing routinely miscredits the framing to Dymitruk. Artifacts: [PDF](https://eventmodeling.org/posts/event-modeling-cheatsheet/cheatsheet.pdf) and [Miro board](https://miro.com/app/board/uXjVOia7ydY=/?share_link_id=194982904636).

- [Great User Experience Demands Event Modeling](https://eventmodeling.org/posts/user-experience-event-modeling/) — 2020 (front matter 2020-04-30), by **Eric Lau**, Senior Solutions Architect at Adaptech Group. Argues Event Modeling is the best available vehicle for UX design, framed around "Context Without Measures is Helplessness; Measures Without Context is Chaos" — context as the situation in which a command may occur, measures as the commands actually available there. Adaptech house material rather than Dymitruk's own.

- [Natural Human Thinking — Event Storming vs Event Modeling](https://eventmodeling.org/posts/human-natural-thinking/) — 2020 (front matter 2020-06-16), by **Rafal Maciag**. A cognitive-science framing mapping big-picture Event Storming to episodic memory and Event Modeling to long-form narrative processing; the site's most explicit treatment of the EM-vs-ES distinction after the About page. Independently uses the **three**-block framing (events, views, commands), corroborating Dymitruk's article against the cheat sheet's four.

### Articles about Event Modeling

- [Why You Need to Know About Event Modeling: An Intro](https://www.linux.com/news/why-you-need-to-know-about-event-modeling-an-intro/) — year unverified, likely 2023, Linux.com (Linux Foundation). Editorial intro naming Dymitruk, CEO and founder of Adaptech Group, as the originator, and describing the method as seven steps with all information represented from the user's perspective. Third-party editorial, not authored by Dymitruk. Page could not be fetched.

### Tools

- [Event-Modeling Specification Language (EMSL) — eventmodeling-toolkit](https://github.com/event-modeling/eventmodeling-toolkit) — last updated 2025-07-02, MIT licensed, under the official `event-modeling` GitHub org. The only attempt at a formal EM notation under the official org. `specification/NOTES.md` enumerates the element set (read model, command, event, UI and Processor swimlanes, Aggregate swimlanes) and the legal connections, which match Dymitruk's connection rule exactly — with no Event→Command edge. `specification/README.md` is currently an empty stub. **Contributor identity was not verified; do not attribute this to Dymitruk.**

---

## Mirrorable on GitHub

These entries have public source on `github.com` / `raw.githubusercontent.com` and can be mirrored verbatim rather than cited by summary. Raw base for the site: `https://raw.githubusercontent.com/event-modeling/eventmodeling.org/master/content/...`

| Entry | Source path or repo |
| --- | --- |
| [Event Modeling: What is it?](https://eventmodeling.org/posts/what-is-event-modeling/) | `content/posts/what-is-event-modeling/index.md` |
| [Event Modeling Introduction (home page)](https://eventmodeling.org/) | `content/_index.md` |
| [About — Event Modeling](https://eventmodeling.org/about/) | `content/about/_index.md` |
| [Event Modeling Traditional Systems](https://eventmodeling.org/posts/event-modeling-traditional-systems/) | `content/posts/event-modeling-traditional-systems/index.md` |
| [Specifying Complex Views (draft)](https://raw.githubusercontent.com/event-modeling/eventmodeling.org/master/content/posts/specifying-complex-views/index.md) | `content/posts/specifying-complex-views/index.md` |
| [Event Modeling Cheat Sheet](https://eventmodeling.org/posts/event-modeling-cheatsheet/) | `content/posts/event-modeling-cheatsheet/` |
| [Great User Experience Demands Event Modeling](https://eventmodeling.org/posts/user-experience-event-modeling/) | `content/posts/user-experience-event-modeling/index.md` |
| [Natural Human Thinking](https://eventmodeling.org/posts/human-natural-thinking/) | `content/posts/human-natural-thinking/index.md` |
| [Resources — Event Modeling](https://eventmodeling.org/resources/) | `content/resources/_index.md` (+ `stickers.svg`, `stickers2.svg`) |
| [eventmodeling.org site source](https://github.com/event-modeling/eventmodeling.org) | whole repo, incl. `data/authors.toml` |
| [dymitruk.com personal blog](http://dymitruk.com/) | [adymitruk/adymitruk.github.com](https://github.com/adymitruk/adymitruk.github.com) |
| [EMSL / eventmodeling-toolkit](https://github.com/event-modeling/eventmodeling-toolkit) | whole repo, `specification/` |

---

## Known gaps

**The 2018 Medium article — no URL located.** Dymitruk has said in interviews (SE Radio 539, Semaphore) that Event Modeling began as a Medium post he wrote in 2018 which reached the Hacker News front page in October 2018, and that he then created eventmodeling.org and moved a cleaned-up version there — becoming "Event Modeling: What is it?" (dated 2019-06-23). Targeted searches on `medium.com/@adymitruk`, `adymitruk.medium.com` and the HN submission returned nothing; the post appears to have been deleted after migration. **No Medium URL should be cited for this without independent confirmation.** It is recorded here as a known-missing item rather than given a guessed link.

**The CACM interview URL is probably not canonical.** Search consistently returns `cacmb4.acm.org`, which looks like an ACM staging or backup host. Explicit searches of `acm.org` domains surfaced no `cacm.acm.org` equivalent, and neither host was fetchable. The year (likely 2022) is inferred from surrounding material, not confirmed.

**Hosts unreachable from the research container.** Egress was restricted to `github.com` and `raw.githubusercontent.com`. Everything on linkedin.com, infoq.com, acm.org, leanpub.com, semaphore.io, se-radio.net, podcast.eventmodeling.org, youtube.com, nebulit.de, adaptechgroup.com, yowcon.com, hanselminutes.com, dotnetrocks.com, linux.com, podbean.com and newsletter.nerdnoir.com was established from web-search titles, URLs and content summaries only. Quotes attributed to those sources should be spot-checked against the pages before being reproduced verbatim.

**Building-block count is genuinely inconsistent across sources.** Dymitruk's "What is it?" says three ("we will only use 3 types of building blocks as well as traditional wireframes or mockups"; "3 moving pieces and 4 patterns based on 2 ideas"), and Maciag's post independently uses three. The Bortz cheat sheet says four, promoting the wireframe/trigger to a first-class block. Any document asserting a count should say which source it follows.

**The four patterns are unnamed in Dymitruk's own writing.** He writes prose sections (Commands, Views, Integration, Translation, Automation) and marks the count only with "We just covered the first 2 patterns of the 4 that are needed to describe most systems." The labels *Command Pattern / View Pattern / Automation Pattern / Translation Pattern* originate in the cheat sheet.

**Adam's hedge on the seven steps is unverified.** A search summary of the SE Radio interview reports him saying the seven steps "are really a guideline"; the page could not be fetched, so this phrasing is not confirmed.

**Adaptech Group's own written material is uncatalogued.** The site could not be fetched, so any whitepapers, case studies or workshop material it hosts are unknown.

**Dates and identities not established:** the .NET Rocks! episode 1928 publication date; the Loosely Coupled Show episode date; the Linux.com article's year; the DDD TW 2021 talk title (reported by search summary only); the InfoQ interviewer's name; the CACM interviewer's name; the EMSL toolkit's actual author.

**Recording availability unconfirmed for:** the YOW! Sydney 2023 masterclass (paid workshop, likely never recorded), the DDD TW 2021 talk, and both Event Modeling Conference keynotes (2025 and 2026).

**Transcripts:** the Event Modeling and Event Sourcing Podcast episode pages carry summaries and timestamped chapter markers; whether full transcripts are published was not confirmed. SE Radio 539's transcript is auto-generated by the publisher's own admission.

**Forthcoming work, not yet a citable source:** as of April 2026 Dymitruk was reported to be writing the definitive Event Modeling book, with a companion booklet due out first. Neither has been located as a published artifact.
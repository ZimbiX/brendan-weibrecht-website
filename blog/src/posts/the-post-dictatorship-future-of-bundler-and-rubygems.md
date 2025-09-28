# The post-dictatorship future of Bundler & RubyGems

2025-09-27

## Background

If you've been under a rock, there's been major chaos over the past week in the land of Ruby. I am extremely concerned, and have been following developments closely. I think [Joel Drapper's posts](https://joel.drapper.me/p/rubygems-takeover/) have been the best and most journalistically thorough representation of the situation. TL;DR: Ruby Central, at the behest of Shopify, performed a hostile takeover of Bundler, RubyGems, and RubyGems.org.

## Motivation for the hostile takeover

Despite Ruby Central funding work on these projects, they had no right to assert ownership of them.

Shopify is now the primary source of Ruby Central funding and seemingly in control of the board. Ruby Central is not a democracy.

What Shopify stand to benefit from this level of control over the core of the Ruby ecosystem is not at all clear. Perhaps power for the sake of it? Perhaps it's DHH removing political opponents from key roles in Ruby. Perhaps Shopify just feel entitled to control it after putting so much money into it.

## Ctrl+Z

I think there's a lot of naivety in [this post from Freedom Dunlao](https://apiguy.substack.com/p/a-board-members-perspective-of-the), a Ruby Central member. Losing all funding would be a lot easier to recover from than this takeover.

The takeover can't even be fully undone. Even if all access & ownership rights are restored to the maintainers, some of the team are so burned by these events that they don't want to come back.

## Public discussion

I am very disappointed this wasn't discussed at the Ruby Melbourne meetup on Thursday. By the end of the talks, without it being raised, I was going to bring it up myself, but sadly we were rushed out the door.

I've seen a lot of writing about what happened, but not much about what a possible future could be, assuming Ruby Central continue to refuse to give full control back to the community. I don't know what the path from here would be, but I think we need to discuss it. I hate that so much of this situation is rumour/speculation. I want conversation in the open. As they say, sunlight is the best disinfectant.

As an outsider (not part of the ex Bundler/RubyGems team), perhaps my thoughts are not helpful. But in an information vaccuum, I'd like some sort of clarity.

## Options for the future

Perhaps Ruby Central will eventually continue the projects, and we'll learn to live under such a dictatorship. But I am of the strong opinion that the Ruby community need to take back control, regardless of whether Ruby Central want us to have it.

### Code

For the code, perhaps the repos must be hard-forked. But then getting everyone to switch to using these alternate releases would probably be very difficult. We'd need to raise awareness of the problem and many people may be too apathetic to move from their status quo.

### Funding

For funding development, perhaps Ruby Together must be reinstated.

There'd need to be a system in place to avoid recurrence of the financial coercion that caused the takeover. Having a lot of smaller donors rather than a few big ones would help with this. And perhaps there's ways to not require as much finance.

### Repository of gems

Dealing with rubygems.org sounds like the most difficult part to me. To create a clone of it, we'd need all of its content (gems and metadata). Without direct access to the database, I think the content would have to be downloaded and scraped. Then there is what I see as the most difficult aspect: giving ownership of each gem to the right people. We need to ensure 

Somewhere in all the news I've read about the situation, I saw that it costs thousands of dollars a month (presumably USD) to run rubygems.org. If this was all for server costs, and not the on-call roster, I do wonder whether there is a cheaper way to host the catalogue - perhaps as static files and in the style of Linux distros' distributed mirrors.

### Spinel & rv

In the longer term, Spinel is exciting. Their project rv looks like it'll be a promising solution for everything except serving the catalogue (rubygems.org), but it looks to be a long way off. I have faith in them though, given it's the ex-Bundler lead and Ruby Together founder; so I've donated and I think I'll set up a recurring donation. I encourage everyone reading this to do the same.

+++
title = "I Was Using AI Tools Wrong and Maybe You're Too"
date = 2024-09-21T11:22:53+05:30
draft = false
cover.image = '/images/tools_wrong.jpg'
showToc = true
tags = ["Opinion"]
+++

> _"This quote is here to trick you into thinking this article is profound."_
>
> \- Me

AI tools like ChatGPT and Claude are basically the new shiny toys every developer has. When they first rolled out, I wasn’t too interested. But since July of last year, things escalated quickly. I went from “meh, I’ll Google it” to “GPT, save me!” in record time.

## The Excitement of AI Assistance

When I first gave GPT a spin, it felt like magic. I still remember the first thing I asked it to do: generate a JSON file for some random questionnaire app I was building while learning Flutter. I gave it a format, and boom! It spat out the entire thing in seconds. My mind was blown. It just saved me half an hour of my life—half an hour that I probably spent staring at it in awe. From that point on, I kept using it to do all sorts of stuff: generating dummy data, plotting graphs, and solving my daily "should I Google this or nah?" dilemmas.

## Where Things Started Going Wrong

At first, I thought I had this AI usage thing under control. I only used it for stuff I could already do but didn’t want to waste time on, you know? But slowly, I started relying on it more and more. Got stuck on an algorithm? Ask GPT. Hit a bug? Ask GPT. Forgot how to breathe? Ask GPT (okay, not quite, but you get the idea). Google who? Stack Overflow what? GPT became my trusty sidekick... until it wasn't.

Turns out, asking AI to do the heavy lifting for you isn't the best idea for your brain. Who knew, right? The more I leaned on AI, the less I actually thought for myself. My problem-solving muscles started to atrophy. Things that used to be simple began to feel like climbing Everest without a sherpa. My first instinct when hitting a snag? “GPT, halp!” It was like outsourcing my brain, but without the vacation perks.

## The False Sense of Progress

One of the worst parts about overusing AI tools is that it _feels_ like you’re making progress. Things are getting done! Boxes are being checked! High-fives all around! But here's the catch: I wasn't actually learning anything. I was just using AI-fed answers without understanding the “why” behind them. When I had to tackle something new or explain what I did, I quickly realized my knowledge was about as deep as a puddle.

## The Turning Point

Things started changing when I started learning Go. I was building CLI tools like Go versions of `wc`, `cat`, `grep`, and reading the book **Writing an Interpreter in Go** (wrote a [blog](https://jitesh117.github.io/blog/things-building-an-interpreter-taught-me/) about it, btw). I was feeling like a Go pro, but then came the real test: backend development.

I was pumped! After watching some of [Anthony GG](https://www.youtube.com/@anthonygg_)'s videos, I jumped into building a REST API project. But as I dug deeper, I quickly realized I didn’t fully understand key concepts like handlers, REST design, or database schemas. Every time I hit a roadblock, instead of figuring things out, I turned to GPT for quick fixes.

This pattern continued in my [code-snippet-manager project](https://github.com/Jitesh117/snippet-manager-go) (which, by the way, is still a work in progress). I relied heavily on GPT for nearly every aspect—from building handlers to setting up the database layer and even Docker configurations. Initially, it felt like I was blazing through development, rapidly getting things done. However, over time, I realized I was only solving the surface-level problems without deeply understanding the underlying architecture.

Eventually, I hit a point where I couldn’t even navigate my own codebase. Whether it was debugging database queries, modifying handlers, or handling Docker configurations, everything started feeling foreign. The overuse of GPT had left noticeable gaps in my understanding, and I had become far too dependent on it to solve problems I should’ve been learning to solve on my own.

For example, I ran into an issue with my login endpoint where user passwords weren’t being processed correctly. After digging through my code and troubleshooting for far too long, I realized the problem was that I was marshalling my structs incorrectly. As a result, the password field was always being sent as empty. Here’s what the marshalling mistake looked like:

![wrong password marshalling](/images/password_struct.png)

This was a fundamental backend issue that I should have caught much earlier. Instead, I was so deep into this AI dependency hellhole that I didn’t even realize I was missing a key part of how struct marshalling works in Go. It was a humbling moment when I had to step back and admit that while GPT had helped me move faster, it had also held me back from truly grasping critical concepts.

At that point, GPT was no longer just a tool—it had become an addiction. I wasn't approaching problems with the curiosity and problem-solving mindset I once had; instead, I was treating GPT like a bible, expecting it to have the solution for everything without engaging my own problem-solving skills.

## Rebuilding Brain Power (AKA Reclaiming My Big Brain Energy)

Admitting there’s a problem is the first step, right? So here’s what I'm doing to restore some brain juice:

1. **30-Minute Rule**: I make myself think for at least 30 minutes before consulting my AI overlord.

2. **Active Learning**: Whenever GPT does bail me out, I make sure to actually understand the code it gave me. Then I tried re-creating it from scratch—without cheating.

3. **Variety is the Spice of Learning**: AI is cool, but I forced myself to mix in other resources like books, docs, and forums to stop my brain from turning into mush.

4. **Challenge Accepted**: I started using GPT to double-check my own solutions instead of writing them for me. Because nothing feels better than proving to a robot that you still got it.

5. **Reflective Practice**: After finishing a task, I’d take a minute to pat myself on the back and think about what I learned, so it would actually stick for next time.

## Healthy AI Tool Usage (Or, How Not to Let AI Own You)

After detoxing from AI, here’s the new set of commandments I follow:

1. **AI is a Muse, Not a Master**: I use AI for brainstorming, not as my personal code slave.

2. **Trust but Verify**: I always check GPT's work against legit sources because even AI can have bad days.

3. **Learn First, AI Later**: Before using AI for a new topic, I make sure to learn the basics the old-fashioned way.

4. **Set Learning Goals**: I make a list of things I actually want to learn, then resist the urge to let GPT do all the heavy lifting.

5. **Enhance, Don't Replace**: AI is great for mindless tasks like formatting or generating test cases, but it’s no replacement for real thinking.

6. **Skill Check**: Every so often, I put GPT in timeout and see how well I can solve problems on my own. Spoiler alert: sometimes it’s not pretty.

## Conclusion

AI tools are like ice cream: fun, exciting, and a little too addictive. But to avoid the brain freeze that comes with overconsumption, you’ve got to use them in moderation. The goal is to become smarter _with_ AI, not dumber _because_ of it.

So, go ahead and use AI, but don’t let it steal your thunder. The key is finding the right balance between taking shortcuts and doing the hard work. After all, the best feeling isn’t when GPT solves a problem for you—it’s when you realize you didn’t need it to begin with.

Just remember, it's more fun to be competent.

## References

- [My blog post on building an interpreter in Golang](https://jitesh117.github.io/blog/things-building-an-interpreter-taught-me/)
- [My code-snippet-manager project](https://github.com/Jitesh117/snippet-manager-go)
- [Anthony GG YouTube Channel](https://www.youtube.com/@anthonygg_)

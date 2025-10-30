+++
title = "Write More Tests, the Pain Is Worth It"
date = 2025-10-30T11:49:26+05:30
draft = false
showToc = true
cover.image = ''
tags = ['tech', 'opinion']
+++

A couple of months ago, I started building the backend for a personal project. Then life happened — work got busy, other stuff took priority, and the project went on pause.

At that time, I skipped TDD because I wanted to move fast. I told myself, “I’ll add tests later.” Classic.

Fast forward to a few days ago, I finally decided to pick it back up. But when I opened the code, I couldn’t make sense of it. I didn’t remember the flow, the logic, or even why I wrote some parts the way I did.

That’s when it hit me — maybe I should write the tests I skipped earlier. It’d help me re-learn the code and make future changes safer. Two birds, one stone.

While writing the tests, I realized how tangled things were. The database layer was tightly coupled with the handlers, so testing anything in isolation was a mess. I ended up refactoring a ton of code just to make it testable.

Painful? Yes. Worth it? Absolutely.

Writing those tests made the code cleaner and way easier to understand. It’s funny — the more tests I wrote, the better the code got.

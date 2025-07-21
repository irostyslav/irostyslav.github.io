---
title: "Vibe Coding intro"
# author: "Rostyslav Ivanitsa"
# authorAvatarPath: "/avatar.jpeg"
date: "2025-07-15"
summary: "Vibe Coding is a revolutionary approach to software development that leverages AI to rapidly prototype ideas by using natural language prompts. Coined by Andrej Karpathy, it allows creators to focus on creativity rather than coding, making it ideal for quick tests and low-stakes projects."
description: "tbd"
toc: true
readTime: true
autonumber: true
math: true
tags: ["pm", "ai", "writing",]
showTags: false
---   

# Vibe Coding Guide

## Introduction to Vibe Coding

![sstable](vibe-coding.jpg#light)
![sstable](vibe-coding.jpg#dark)

Imagine you have a groundbreaking product idea—you can see every detail, and you just know it'll disrupt the market. Then reality hits. You get bogged down by Jira tickets, sprint planning, developer timelines ("this will take two months"), and designers juggling other projects. By the time your concept, riddled with compromises, turns into a clickable prototype for testing, you’ve almost forgotten why you believed in it.

Now, you can cut this process down from weeks to mere hours.

Enter **Vibe Coding.** Want more background? Check out [Simon Willison’s deep dive on vibe coding](https://simonwillison.net/2025/Mar/19/vibe-coding/) and [Karpathy’s original tweet](https://twitter.com/karpathy/status/1886192184808149383) that ignited the trend.

The idea is simple. You’re no longer engineering or writing detailed specifications. Instead, you become a creative director for an AI. You use natural language to prompt the system with your vision. For example, “Create a Tinder-style swipe feed for kittens” or “Add a button with a fiery animation here.” The astonishing part? It works.

The term "Vibe Coding" was coined by Andrej Karpathy, who described it as a state where “you fully immerse yourself in the vibe and forget code even exists.”

This isn’t just another framework—it’s a paradigm shift. It's about removing the usual bottlenecks—approvals, resource requests, and endless status updates. Now, you or your designer can work directly with a large language model (LLM) to assemble a working prototype.

But, of course, there’s a catch—a significant one. While Vibe Coding speeds up prototyping, it comes at the expense of quality. The generated code can be difficult (or even impossible) for developers to maintain and scale. Issues such as security vulnerabilities, bugs, and massive technical debt are common downsides.

The decision comes down to this trade-off—quick prototyping versus predictable technical solutions. It ultimately depends on how you integrate Vibe Coding into your workflow. Interestingly, the bottleneck has now shifted. It's no longer the development or design process—it’s about finding and solving *real* user problems. And that’s where product management takes center stage.

---

## Vibe Coding vs. Development with AI and LLM

Using AI development tools doesn’t automatically mean you’re doing Vibe Coding. Here’s the dividing line:

At its core, Vibe Coding is about forgetting the existence of code entirely. You fully rely on the AI, engaging with little to no manual intervention.

Karpathy originally described this with examples like:

- “I describe what I want and rarely touch the keyboard.”
- “I always press ‘Accept All’ and don’t check the code changes.”
- “If errors pop up, I paste them into the chat with no further explanation—and it usually solves them.”
- “When the AI fails to fix something, I ask it to make random changes until the bug disappears.”

This approach is experimental and works best for weekend passion projects and quick idea tests. The crucial distinction is that you don’t engage deeply with the code.

For more on this distinction and when vibe coding makes sense, read [Not all AI-assisted programming is vibe coding (but vibe coding rocks) by Simon Willison](https://simonwillison.net/2025/Mar/19/vibe-coding/), or check [Addy Osmani's field guide to responsible AI-assisted development](https://addyo.substack.com/p/vibe-coding-is-not-an-excuse-for).

### What’s Not Considered Vibe Coding?

Professional-grade development with AI.

Being a professional developer involves much more than writing code. Your work ensures the product is:

1. Reliable and stable.
2. Secure.
3. Understandable to other developers.
4. Easy to maintain and scale.

The typical output of Vibe Coding doesn’t meet these criteria—at least, not yet.

If you’re reviewing, testing, and understanding the logic behind AI-generated code to ensure its quality, that’s **not** Vibe Coding. That’s efficient, AI-powered development.

That said, Vibe Coding has its place. While it may not be suitable for large-scale software development, it's ideal for certain tasks, such as validating ideas and developing simple prototypes. Knowing how to use it effectively is key.


---

## Tools for Vibe Coding

Here are some tools making waves in the Vibe Coding space, along with their strengths, limitations, and links to more resources:

### 1. **Bolt**

Bolt is a browser-based, AI-powered tool for creating full-stack web applications in JavaScript. The interaction happens entirely through a chat interface. Instead of writing code, you describe what you need, and Bolt makes live changes while offering real-time previews.

- Deep dive: [Bolt AI Website Builder Tutorial (No Code MBA)](https://www.nocode.mba/articles/bolt-ai-website)
- Full guide: [Bolt AI App Builder: The Ultimate Guide](https://www.nocode.mba/articles/bolt-ai-new-guide#)

#### Key Features:

- No setup required—everything runs in the browser.
- Integrations with key services like [Supabase](https://supabase.com/), Netlify (hosting), Stripe (payments), and Figma (design imports).

#### Drawbacks:

- **Limited functionality:** Bolt is excellent for scaffolding and basic MVPs but struggles with non-standard business logic or legacy integrations.  
- **No manual code editing:** When Bolt gets stuck on an issue, you can’t tweak the code directly—you'll either have to adjust your prompts or export the code to another environment.
- **Focus on web applications:** It doesn’t currently support mobile apps or browser extensions.

#### Pricing:

Starts at $20/month with a free plan offering basic functionality.

### 2. **Cursor**

If Bolt is a playground for prototyping, Cursor is the professional option. This intelligent IDE (Integrated Development Environment) supports robust software projects and understands your entire codebase.

- More info: [Cursor: Official Site](https://www.cursor.so/)
- Tutorial: [Cursor AI Tutorial for Beginners](https://www.youtube.com/watch?v=3289vhOUdKA&ab_channel=VoloBuilds)

#### Key Features:

- Contextually aware responses based on your existing code.  
- Allows you to rewrite, refactor, or generate new code without switching to a separate chat interface.
- Accepts documentation links for better AI understanding.

#### Drawbacks:

- **Steeper learning curve:** Cursor doesn’t generate complex systems from scratch; it speeds up repetitive tasks but leaves architecture and logic to you.
- **Desktop-based:** Unlike Bolt, Cursor requires local setup and dependency management.

#### Pricing:

Starts at $20/month after a two-week free trial.

### 3. **Lovable**

Lovable bridges the gap between Bolt’s simplicity and Cursor’s complexity. It generates full-stack applications via chat-based interfaces but encourages a more structured, iterative approach.

- Getting started: [Lovable Documentation](https://docs.lovable.dev/introduction/getting-started)
- Tutorial: [Lovable AI Guide (DataCamp)](https://www.datacamp.com/tutorial/lovable-ai)

#### Key Features:

- Supports React-based apps with integrations for Stripe, Supabase, and Netlify.
- GitHub integration for seamless version control.

#### Pricing:

Free plan for up to five daily prompts; the Pro plan starts at $25/month.

### 4. **Replit**

Replit is perfect for workshops, internal training, or proof-of-concept development. It’s an all-in-one environment where you write, deploy, and host your app without additional services.

- Course: [Vibe Coding 101 with Replit (DeepLearning.AI)](https://www.deeplearning.ai/short-courses/vibe-coding-101-with-replit/)

#### Key Features:

- Instant deployment—write your code, hit "Deploy," and your app goes live.

#### Pricing:

Free plan available; premium features start at $20/month.

### 5. **v0 by Vercel**

v0 focuses exclusively on UI development. It’s perfect for front-end designers looking to create polished visual interfaces with minimal effort.

- Overview: [v0 by Vercel](https://v0.dev/)
- Video walkthrough: [Vercel's VP of Product on how to use v0 (YouTube)](https://www.youtube.com/watch?v=sCFS_U7d9Do&ab_channel=GregIsenberg)

#### Key Features:

- Generates UI designs based on textual descriptions or images.

#### Pricing:

Offers free monthly credits; Pro plans start at $20/month.

### 6. **Windsurf**

A direct competitor to Cursor, Windsurf is a downloadable IDE with some unique features, such as the ability to cascade changes across multiple files based on higher-level instructions.

#### Pricing:

Generous free plan; premium access starts at $15/month.

### 7. **a0.dev**

Tailored for mobile app development, a0.dev focuses exclusively on creating React Native apps for iOS and Android.

#### Key Features:

- Cloud-based builds for iOS apps—no need for a Mac to test or publish

## Final Thoughts

Vibe Coding represents a bold new era in software development. It’s fast, intuitive, and democratizes the prototyping process. However, its limitations—especially in terms of scalability and maintainability—mean it’s not a one-size-fits-all solution.

By understanding its strengths and weaknesses, and by tapping into expert guides and community resources, you can incorporate Vibe Coding effectively into your workflow, saving time on testing ideas while reserving full-scale development for when it’s truly needed.

**External Resources & Further Reading:**

- [Simon Willison’s Weblog: Not all AI-assisted programming is vibe coding (but vibe coding rocks)](https://simonwillison.net/2025/Mar/19/vibe-coding/)
- [Addy Osmani on Substack: Vibe Coding is not an excuse for low-quality work](https://addyo.substack.com/p/vibe-coding-is-not-an-excuse-for)
- [Lovable Documentation](https://docs.lovable.dev/introduction/getting-started)
- [Bolt AI Website Builder Guide and Tutorials (No Code MBA)](https://www.nocode.mba/articles/bolt-ai-website)

Explore, prototype, and experiment—but vibe responsibly.
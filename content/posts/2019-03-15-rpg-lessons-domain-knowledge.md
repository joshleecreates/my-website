---
author: Josh Lee
categories:
- For Developers
date: "2019-03-15T12:28:52Z"
excerpt: Planning is essential. Plans are useless.
tags:
- Lessons from the Tabletop
- featured
title: 'Lessons from the Tabletop: Know Your Domain'
url: /rpg-lessons-domain-knowledge/
---

 *This is Part III in my series* Lessons from the Tabletop: Things I’ve learned about project management from Dungeons &amp; Dragons*. [Read Part I](https://joshuamlee.com/project-management-rpg-lessons/). [Part II](https://joshuamlee.com/lessons-tabletop-letting-go/).*

Many dungeon masters fall victim to the trap of over-preparation.

Spending a lot of time preparing a specific plan for a game can make it difficult to let go of that plan (as I wrote in [Part II](https://joshuamlee.com/lessons-tabletop-letting-go/)). One might conclude that planning is a waste of time. As an absolute, this could not be more wrong.

> “Planning is essential. Plans are useless.”

Rather than preparing for one story, a good dungeon master prepares for any story.

These days, I need only prepare a few bullet points for any specific session. The rest of my preparation is the accumulated library of material that I have gathered. I can use this material in any combination to guide a story.

This allows me to say “yes and…” to my players more often, building on their ideas rather than squashing them.

- - - - - -

When we turn to building software, our goal is to quickly adapt, providing solid solutions to whatever problems arise. In this context, our “library” of material (knowledge) is two-fold: domain knowledge, and technical knowledge.

Technical knowledge is the clearest. It is our knowledge of the tools and patterns that we will use. For example, when using a framework, do we know it well enough to extend existing features instead of reinventing the wheel? Do we know how each feature will scale?

Domain knowledge is more vague, and more often ignored by technical team members. It is no less important though. This is the knowledge of the real-world objects and systems that our software models.

In any project of significance, we will find many domains. These may or may not interact and overlap. For example, consider a simple e-commerce application. For this example we will only have one product for sale, and no shopping cart.

*E-Commerce* sounds pretty specific for a domain. But when we break it down, we’ll find that there are many subdomains all involved in selling a single item. On a first pass we identify: financial transactions, marketing analytics, shipping.

We could break this down even further. Shipping depends on a system of knowledge for addresses &amp; locations. We’ll also need a way for handling dates, times &amp; durations.

Smarter people than me have written about how to model the real world with domain driven design. That is not the point of this allegory.

Once we have broken down the project domains, it is our job to understand them as well as we can. Most projects will involve both domains we are familiar with and some that are new to us.

If no project ever added new ideas, we would never grow as developers and nothing novel would ever be built. On the other hand, if we attempt a project with no knowledge of any of the relevant domains and tools, we will be doomed to flounder.

The deeper we understand something, the more we can expand upon it or integrate it with something else. The more of a business or project that we understand, the easier it will be to add innovations.

Have a plan from which to deviate.

**This is Part IV in "Lessons from the Tabletop: Things I've Learned About Project Management from TTRPGS"**:

 - [Part I: Align on your goals](https://joshuamlee.com/project-management-rpg-lessons/)  
 - [Part II: Letting go](https://joshuamlee.com/lessons-tabletop-letting-go/)  
 - [Part III: Know your domain](https://joshuamlee.com/rpg-lessons-domain-knowledge)
 - [Part IV: Sharing the Narrative](https://joshuamlee.com/sharing-the-narrative)

# Agent Style

This repository contains my personal preferences, recorded so AI applications and agents can approach tasks in a way more similar to how I prefer to do them or have them done.

This is not an attempt to define a best-practices guide for either writing software or conducting science.

## What's here

* **`AGENTS.md`** is the short operational version intended to be given directly to coding agents. It focuses on the relatively small number of preferences that most strongly distinguish my style from an agent's normal defaults.
* **`CODING_STYLE.md`** is the longer reference. It contains additional detail, rationale, examples, and preferences that would be unnecessarily expensive or distracting to include in every agent context.

`CODING_STYLE.md` is the fuller description of the style. `AGENTS.md` is a deliberately compressed representation intended for routine use.

## How was this made?

I pointed ChatGPT at my GitHub and said, basically, "Tell me my style."

This was terrible.

Its main choices to focus on were repositories it thought were modern and had a good style, like `snoot` and `repo-riposte`. The problem was that those repositories were themselves agentically coded.

So I set about making a long list of my own preferences in coding, then asked ChatGPT to take those as a starting point and look through my older repositories to see if it could improve the list.

This was materially better, but I would say it still felt like it was 80% just a normal Python style guide, 10% things that were fairly specific to my preferences, and 10% things that were wrong.

The next thing I did was take the long style guide ChatGPT had produced and, for each point, give it feedback. I said whether it felt like me or not, and why.

Then I had ChatGPT generate a new guide based on that feedback, and it looked much better.

I believe those feedback notes gave it more insight into the details behind my decisions, while the broader conversation gave it enough context to understand how my personal style differs from generic coding practices.

So did I write this, or did AI write this?

`¯\_(ツ)_/¯`

## Why publish this?

Coding agents already know a great deal about programming. I am generally not interested in repeating conventional software-engineering advice that they already understand.

What I *am* interested in is specifying the places where reasonable developers make different choices.

Things like:

* how much abstraction is desirable;
* when convenience helpers are worth adding;
* how APIs should expose complexity;
* how packages and repositories should be organized;
* what belongs in tests;
* how scientific provenance and reproducibility should be handled;
* when code should be explicit rather than clever;
* what kinds of architecture feel natural or unnatural.

Those decisions accumulate into something resembling an engineering style.

## Make your own, please

You are welcome to use these instructions directly if you genuinely want an agent to work according to my preferences.

But the larger point of this repository is to suggest that your AI applications and agents may produce better work for you if you write down your own style.

So copy this if it helps. Delete most of it. Replace my preferences with yours.

Or, better yet, start from scratch and see what comes out.

The useful artifact at the end should describe **you**.

## Using it

For tools that understand the `AGENTS.md` convention, the root `AGENTS.md` can be used directly.

It can also serve as a source for whatever project-level or global instruction mechanism a particular coding agent supports.

The longer `CODING_STYLE.md` is primarily a more comprehensive reference for situations where additional context is useful. I don't expect it to be required reading for every agent invocation, but it might be kind of nice to include when asking an agent for a big batch of boilerplate if you want it to look like my boilerplate.

Because these preferences will evolve, the repository history and tagged releases can also serve as snapshots of the style at a particular point in time.

## What's next?

I think a writing style guide and a scientific method style guide would also be useful.

## License

I love Apache 2.0, so it is a big deal for me to use some other license.

In this case, though, there is something indelibly personal about authoring my engineering and scientific style, so attribution feels natural. The written material in this repository is therefore licensed under the **Creative Commons Attribution 4.0 International license (CC BY 4.0)**.

I want people to be able to use this material for essentially any purpose: personal or commercial, unchanged or heavily modified, as instructions for an agent, as inspiration for another style guide, or as raw material for something I did not anticipate.

The main requirement is attribution, including indicating when you have modified the material.

See `LICENSE` for the full license terms.

However, I would rather see you make your own repository from scratch and license it any way you want. I want to see how unique your own style guide can become, because I am sure there are lessons in it that I have never thought about.


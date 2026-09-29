+++
title = "Search and Thoughts on LLM-coding"
description= "LLM will not make buggy software. You will."
date = 2026-09-07

[taxonomies]
tags = ["LLM", "Software Development"]
+++

## Intro:
Holy moly. What a year for software developers. With LLM coding IDEs, I see why some LLM hypists say "programming is solved" after all those agents complete coding tasks with blazing speed. However, even as a college kid, I feel that LLMs are rather a double-edge sword than a pure productivity tool.

During my last internship, I saw Claude Opus in Kiro followed good software design patterns that I would not have implemented on my first try. Claude also explained to me about how an application that I worked on would be deployed into a dev cluster environment. Most importantly, the steering documents and skill Markdowns controlling the agent were written by experienced software developers, including my former manager. 

However, LLMs could easily reveal human incompetence when used by careless people. When vibe-coders (like me in first/second year oops) use LLMs, they would not be able a secure and reliable software becuase they have little codebase understanding and debugging skills.

Even during the pre-LLM era, [Microsoft](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/ownership.pdf) researchers investigated the relationship between code expertise, pre-release defects and post-release faliures. With linear regression models controlling for code attributes, the researchers found that top major contributors have a statistically significant negative relationship with defects while minor devs have a statistically significant positive one. Codebase understanding matters to avoid a debugging nightmare. Needless to say, software development is exactly the expertise that requires codebase understanding and software troubleshooting.

This experience and study changed my thoughts on LLM coding. LLMs can amplify expertise and ignorance. LLM coding should absolutely be part of modern software development, but outsourcing implementation can also mean outsourcing understanding. 

Now that I hope I have convinced you of the significance of both LLM-coding IDEs and software development expertise, I would like to step a bit further. Since my first year, I have been questioning the actual effect of LLM coding on technical understanding and work performance. My biggest concern is that LLMs large generation bandwidth exceeds human's comprehension bandwidth. 

So I started looking into two questions:

Does LLM coding hinder or enhance technical understanding?

And does LLM assistance make production code harder to maintain?

## LLM coders are cheaters, or are they?

[Anthropic](https://www.anthropic.com/research/AI-assistance-coding-skills?target=_blank) conducted an interesting randomized experiment this January in this topic. They recruited 52 (mostly junior) software engineers with a good exposure to Python and LLM-coding assitance but had not used a target Python library, Trio. In the study, the engineers had to complete a warm-up coding task, two Trio coding tasks related to asynchronous programming, and a quiz. Each of them needed to complete their task within a dedicated online coding platform with an AI assistant in the side bar. 

![Anthropic model usage.](/assets/llm-coding/anthropic.jpeg)

Unsurprisingly, the groups who used LLM to code averaged 50% on the quiz while those who hand-coded averaged 67% (Stop moving your hands...). This result is a statistically significant 17% difference with a fairly large effect size (Cohen’s d=0.738). The largest gap in scores between two groups on debugging, suggesting that LLM marginally hinders our understanding code failures. The low-scoring groups either delegated all of their thinking and work to LLM or asked LLM to debug repeatedly without asking clarification questions. The high-scoring groups, on the other hand, kept asking clarifying questions to LLM. 

However, the most interesting part of this study is the pattern of highest scorers. These participants were not the manual coders, but LLM-coders who asked insightful follow-up questions. They were the slowest compared to other participants but some of them scored 86% while some LLM-coders with little understanding scored 39%. This significant score gap shows the way an engineer interacts with this tool while attempting to be efficient affects technical understanding the most.
 
This finding further validates my perspective on LLM-coding where LLM acts as an amplifier of our ignorance or expertise. LLM coders are not frauds. Vibe-coders are. Vibe-coders do not pay attention to codebase, let alone understand what each piece of code does. 

Beyond structural understanding, good software devs also need to evaluate LLM's code against real-world deployment standards. We do not live in the era where writing a working piece of code is valuable anymore. We need to focus our shift to how to maintain LLM-generated code in a production environment with multiple users. I investigated further on LLM

## 'LLM's codes are not maintainable' may be incorrect.

One of the best practices in software development is maintainability. The maintainable codebase allows new devs to grasp its behavior easily, develop new features, and reduce technical debts. I found two interesting studies in this topic. One revealed that the maintainability problem with LLM-generated code is not the code itself, and another revealed that the increased generation lets developers create more code than they can supervise and understand. 


[This research](https://link.springer.com/article/10.1007/s10664-026-10889-1) found that LLM-generated code may not inherently contain systematic maintainability disadvantages compared to manual code on feature-level. The researchers measured their maintainability of both LLM-generated and manual implementations by CodeHealth (CH) Score that penalizes code smells. Surprisingly, the Frequentist result of CH Score showed no significant difference, and the Bayesian result revealed a CH Score improvement when the original solution was written by LLM coders. 

![Echoes of AI paper experiment](/assets/llm-coding/echoes_of_AI.jpeg)

Researchers gave several reasons for this finding but what stood out to me were the following two. First, they speculated that LLM tended to homogenize code toward standard, idiomatic language constructs. This type of application code is beneficial for maintainability because developers can imagine a correct abstraction of its behavior. Second, LLM's code had fewer linting warnings. This results was primarily because LLM-coders used a more functional style with fewer method boundaries. The code's idiomatic simplicity improves readability, benefiting both human comprehension and LLM interpretation and potentially lowering the barrier to contribution for new developers.

Of course, there is a study with bad news — a larger LLM-generated codebase can have many technical debts. In January 2026, Carnegie Mellon University researchers published [a very rigorous peer-reviewed study](https://arxiv.org/pdf/2511.04427) that compared open-source projects adopting Cursor to the similar projects that did not adopt it. They identified 800+ repositories adopting Cursor between January 2024 and March 2025 and used difference-in-difference model to compare against 1300+ similar repositories that did not use Cursor. 

![CMU paper result](/assets/llm-coding/CMU.png)

In the first month after Cursor adoption, the model revealed **281.3%** increase in lines added on average. Over the full post-adoption period, SonarQube static analysis showed an average 30.3% increase in warnings (across reliability, maintainability, and security) and a 41.6% increase in cognitive complexity. To make matters worse, these warnings persisted beyond the adoption period, contrasting the transient velocity gains. While some warnings might not affect product quality, this study underscores the high volume of code generation can consistently increase the future maintainance burden of contributors. 

From these two studies, it is safe to conclude that LLM-assisted code is not inherently less maintainable but has a potential to be burdensome in proportion to codebase size. Controlled experiments in the first study have found little or no downstream maintainability penalty. However, repository-level evidence in the second study gives us that adoption of coding IDEs can substantially accumulate technical debts and static-analysis warnings by trading off long-term maintenance burden for initial productivity gains. As the system complexty grows, the security teams expand their verification of those warnings and are likely to suffer from maintainance burden, although their work may also be assisted by frontier models to identify and patch vulnerabilities.

## My own thoughts on LLM coding
I have actually become more positive about LLM coding from these studies. We can leverage LLMs to code, but we must also strive to understand the generated code and fix its issues. However, all studies above have one catch — the participants are still professional software developers. They were very likely to recall and apply their technical knowledge during experiment. Many software developers, including me, are still novices with insufficient knowledge or experience to get hired (We don’t have 5 years of claude code experience). For novices, I believe that there are no better ways than manual writing to enhance their technical understanding.

Manual writing should be a primary method to catch up with new practical software development knowledge because it forces active recall. Manal writing is different from copying and pasting someone's words verbatim. Manual writing is the process of recalling your learnings and expressing your own thoughts by yourself. Manual writing slows down during learning so that comprehension can catch up. Only after understanding, LLM delegations becomes an accountable action.

There are multiple ways of manual writing, but the most relevant ones in tech are manual coding and manual documentation. Manual coding helped me understand software knowledge such as:
- APIs: json vs jsonl, rate limiting, idempotency, retries, exp backoffs, Web Sockets, REST, SSE, and gRPC.
- CRUD logic flow: api endpoint setting, loose coupling from business logic, database operations, SQL/ORM.
- UI designs with HTML/CSS.
- Data pipelines: Batch processing, stream processing, backfills, scheduler setup, and concurrency.
- OAuth 2.0 workflow: PKCE, state, code exchange, authentication vs authorization, and CSRF
- Parallelism: latency hiding, blocking/non-blocking message passing implementations.

In case I do not have time to manually code and test, I strive to write good documentation after finishing a project prototype. Specifically, I write:
- What and why a certain tech stack is used.
- What software design is used for core features.
- If exists, what system design patterns are used.
- File and folder structure
- CI tool local setup
- CD setup
- Cloud service setup
- Cache analysis: cache lines, cache-warmup, coherence protocol, eviction policy, CDN, and access pattern.

Some of you may claim that manual typing is the waste of time — LLM will correct itself and improve itslef and resolve issues. Sure, but here are some questions for you:
1. What if LLM made suggestions and changes outside of your expertise? You keep asking follow-up questions? Can you evaluate LLM’s changes effectively?
2. Codebase generated by LLM agents can exceed your cognitive bandwidth. Are you still able to maintain sanity and motivaiton to understand their code? For a long-term project, are you able to retain your knoweldge 2-3 months later? 
3. What if LLM decided to write a subtle unreliable code leading to some bugs? Are you able to debug those issues?

If your answer is no to any of the above questions, code is not the interface If the answer to all three becomes “ask the LLM again,” then code is no longer really the interface between you and the model.

YOU ARE THE INTERFACE. 

You are responsible for translating requirements into instructoins, generated implementations into reliable systems, and failures back into information the model can use.

To avoid this, we have to slow down and understand things. I do not believe passively reading model's outputs strengthens our technical understanding. Understanding takes time. Relying on LLMs will not perfect our understanding. Without deep technical understanding, we cannot become a good developer. Suffering and uncertainties that manual typing give us will make us a more resillient learner as well. Again, understanding takes time.

## Future steps
From the studies above and my experience, I feel the following moves still matter as a software dev:

Common:
- Using a coding IDE is non-negotiable for productivity, particularly ongoing hardware optimization efforts are very likely to lower LLM inference costs.
- Keep classifying each software dev skill to the one that can atorphy and that cannot atorphy. For example, syntax memorization skill can atorphy but system design and debugging skills must not atorphy.
- Documentation is also a must for every project you care about.

Personal dev:
- For learning fundamentals and learning projects, manual typing is the only way for active recalls.
- For projects, focus on development work for deployment, deployment configuraitions, and understanding the codebase eventually. Everybody can vibe-code a demo, but few can make a good product.

Work:
- Spend a good chunk of time and effort authoring/researching good markdowns to polish skills.md, agents.md, and markdown component files. Make sure LLM can generate code that matches your development and deployment standard.
- Always ask follow-up questions 2-3 times at least for the generated codebase. Never forget to document them.


## Debrief
Thank you for reading this blog all the way. 
I admit that this review is not comprehensive enough and outdated. I did not touch security at all, and I have never touched claude code. I intentionally avoided those parts as it is out of my expertise. But I hope you find it entertaining!

All words in this post are manually written by me. 

I hope your technical expertise shines in this pinnacle of LLM era.

Feel free to reach out to me if you have any opinions.

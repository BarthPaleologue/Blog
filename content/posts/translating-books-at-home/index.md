---
title: "Translating Books at Home"
subtitle: "Technical & ethical overview of a modern machine translation project"
date: 2026-07-19T13:57:46+02:00
draft: true
author: "Barthélemy Paléologue"
description: ""
categories: []
tags: []
bigimg: []
comments: true
---

## We are doing translation now? 

In recent months, my partner and I have been reading the english translation of "Legend of the Galactic Heroes" (LOGH) from the japanese original by Yoshiki Tanaka.

While I usually avoid reading books in english as it is not my native language, this translation felt effortless to read.

I was about to discover for myself how translator-dependent the ease of reading was!

In the cas of LOGH, the first 3 books were translated by Daniel Huddleston, and next 3 were translated by Tyran Grillo. (There are 4 more books afterward, but we don't care about them today).

Once my partner reached the 4th book, she shared with me some passages that she found very hard to understand. Pronouns reference were unclear, some sentences had contradictory meaning in the canon of the universe. I felt there was a real risk she would stop reading the book as the friction drains her motivation day after day. 

{{< callout warning >}}
I don't want to throw shade at Tyran Grillo here, there is no telling how much time and money he was given to translate those 3 books, and even the most talented translator would fail under inferior working conditions. Any attempt at making people understand each other better should be celebrated.
{{< /callout >}}

What I am more interested in today is: what can you do in modern days when faced in this situation? 

Faced with this problem, I felt like I had 3 options:
- find another translation
- learn japanese to read the source material
- create a new translation from scratch

{{< callout info >}}
This article will talk about LLMs and the ethics of using them. The article itself was painstakingly written by yours truly (a human), and proofread by both humans and LLMs for correctness and idiomaticity of some sentences. I hope you will find it interesting, no matter what your opinion of AI is.
{{< /callout >}}

## Human translation efforts

The easiest solution by far would be to find another translation that we would find suitable. From what I found, there is no other english translation for book 4 to 6.

There was [a community-led project](https://legendofgalacticheroes.blogspot.com/) to translate the books but it seems they only got around to translate the first one, before sadly stopping in 2015.

I also checked for translation in my native language (French), and found [a crowdfunding project](https://fr.ulule.com/les-heros-de-la-galaxie/)! However the first book is set to release in 2027 so the 4th book is likely a few years away, and I won't wait that long.

## Learning japanese?

Well I did take some japanese classes in high school and at engineering school, but it doesn't go much further away than "Watashiwa Baruterumidesu, yoroshiku onegaishimasu. O genki desuka?". I already have a full-time job and [making a video game in my free-time](https://cosmosjourneyer.com), I can't find time to play music so me finding the time to learn an entire language is unlikely.

## Making a new translation

Our list of options is growing thin. Looks like we will need to make a new translation, or stop reading the books.

### Hiring a human professional

The gold standard for making a new translation is to hire a professional to do it for us. But I am only a software engineer and wallet is definitely finite. It seems hiring a professional english translator costs [between 0.12$ and 0.16$ per word](https://pen.org/report/fairness-in-publishing). Given 3 books of 200 pages, averaging 90k english words per book (270k words total), we can estimate the cost at `270k * 0.14 = 38k$` for those translations.

![combien, meme from La Cité de la Peur (How much ???! in english)](combien.png)

You see I don't have that much money, and if I did I would likely use it for something else than re-translating books!

Seems like human translators are out of the menu for today unfortunately. Let's see if we can find another solution.

### Google translate

When learning english at school, Google Translate was the go to option for students who didn't want to put effort into their work. It is free of course, and notoriously bad for anything serious unfortunately. Funnily enough, they recently tried shoving Gemini inside, [which made the tool vulnerable to prompt injection attacks](https://winbuzzer.com/2026/02/10/google-translate-gemini-prompt-injection-vulnerability-xcxwbn/).

For a real translation project, we will need a system that can take decisions and weigh translation trade-offs when the mapping between source and target languages is not obvious. Google translation is not that system.

### Large language models with reasoning

#### Some context

Large Language Models (LLMs) are neural network trained to predict tokens (pieces of words) given a text context. Modern systems use them to generate the next word in a sentence repeatedly, creating coherent sentences.

By itself, this is not enough for translating entire books: the model will simply generate what's most probable, which is average internet slop text. A translator does not predict the most probable translation, they think about different possibilities, make drafts and more. We pay professional translator precisely because they will produce something better than average thanks to their expertise.

Technological progress has not stopped though, and in late 2024, [OpenAI introduced the first LLM capable of complex reasoning](https://openai.com/fr-FR/index/introducing-openai-o1-preview/) and in context decision making: o1. Reasoning capabilities lead to major improvements across all tasks, culminating in the recent [refutal of Erdos conjecture about the unit distance problem](https://openai.com/fr-FR/index/model-disproves-discrete-geometry-conjecture/). 

These models are capable of planning and adaptation to novel situations, which is exactly what we need.

#### Cost estimate

Earlier, we dismissed human translation earlier based on cost so it would be unfair not to the same for the machine-based approach.

We want to translate 3 books, which we estimated will contain ~270k english words once translated. LLMs do not operate on words directly, instead words are split into tokens. In our case [english words are split in 1.3 tokens on average](https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them), so will need to generate *at the very least* 360k tokens. How much would that cost?

Depending on the size of the model, depending on caching, you can get widely different prices. Let's use 2 extremes of the modern landscape: Deepseek V4 flash which is ok-tier and insanely cheap, and Claude Fable 5 which is very smart and insanely expensive.

| Model | Price per million output tokens | Price for 360k output tokens | Source |
|---|---:|---:|---|
| DeepSeek V4 Flash | $0.28 | $0.10 | [DeepSeek pricing](https://api-docs.deepseek.com/quick_start/pricing/) |
| Claude Fable 5 | $50.00 | $18.00 | [Claude pricing](https://platform.claude.com/docs/en/about-claude/pricing) |

The first time I ran this numbers I was surprised to see how cheap it is (even for Fable)! But let's be careful: we are considering only the output translation tokens. 

As we said, a translator must read the source material (input tokens), read what he already has translated (more input tokens), and most importantly: think, draft and make decisions (output reasoning tokens).

For a large task executed in one go, input tokens are cached and therefore their price is negligible compared to the output tokens. What matters then is estimating how much reasoning will be needed.

As a first guess, let's say a translator spends 99% of its cognitive effort thinking about how to translate, weighing trade-offs, drafting and making decisions, while the remaining percent is used to produce the final result. Instead paying for 360k tokens, we would be paying for 36M tokens!

{{< callout note >}}
Why 99% you may ask? Why not 99.999% or 98%? ~~It that it was revealed to me in a dream!~~ (Joke aside it just felt about right given my experience using coding agents at work where I estimate only 1% of my tokens are used to write the final result itself). You will see at the end that my estimate was shockingly close.
{{< /callout >}}

Let's 100x the previous prices to account for reasoning:

| Model | Price per million output tokens | Price for 36M output tokens |
|---|---:|---:|
| DeepSeek V4 Flash | $0.28 | $10 |
| Claude Fable 5 | $50.00 | $1800 |


Even then, the estimated price is far below what the professional translator would cost us. But there is no telling how the quality will compare at this point. I am curious and it is affordable enough that I want to try and see for myself how far we can take it! 

## Translation Ex Machina

TLDR: If you don't care about the technical details (aren't you curious?), you can find a link to the AGPL licensed repository at the end of the article ;)

### Mapping LLM weaknesses

Let's get inside the engineering part of the project. How will we tackle this problem? Do we just paste the japanese source inside ChatGPT and tell it to "Translate in english, make no mistake!".

![obligatory make no mistake joke](./makeNoMistake.png)

That's of course not what's going to happen, I wouldn't be writing a blog post about it otherwise.

Our goal here is to capture they style of Daniel Huddleston that made the first 3 books so effortless to read and use that to translate the next 3. 

That means using the japanese versions of the first 3 books as well as their translation and the source material from each target book. That's 7 books we need to use for each translation! 

Let's suppose the japanese source contains as many tokens as the english translation. That means each book is equivalent to 90k english words, so 120k tokens per book and we want to fit 7 inside the context.

That's 840k tokens, and the LLM hasn't even started to think, which we estimated before would represent 99% of the token effort.

For context (pun intended), the state of the art of LLM context size is 1M, and you will often see lower values like 256k instead.

As you can see, we need to put in some human thinking here to make this translation happen. The key will be give enough context to make a quality translation, without resorting to dumping all 7 books on the LLM.

## Humanmimicry

Good engineers always try first to find an existing solution to their problems before reinventing the wheel (we do love reinventing the wheel though).

Often nature can show us the way (biomimicry). Evolution already spent hundreds of millions of years finding solutions to problems we care about (think bird inspired wing design, brain inspired artificial neural network or and colony optimization algorithms).

So let's take a look at nature to see how translation is solved. After hundreds of millions of years, natural selection and mutation produced the human translator. We will be looking at how they work to come up with a solution to our problem. 

How does a translator produces a quality translation?

The first observation is obvious: a translator does not work on the entire book all at once. By focusing on a smaller passage, we make sure we have the cognitive space required to make the right decisions.

Ok, but how do we handle long range consistency then?

For human translators, consistency comes from recalling earlier translation decisions. Those decisions can range from a consistent translation for a character name to more conceptual decisions about the style to adopt in a given context.

Right, but isn't there a risk that a mis-translation early on have cascading consequences later in the translation?

Human translator will proof-read their ongoing work and reference again the source material to double check pending decisions. Once the proof-reading yields diminishing returns, go translate the next passage.

Given what we observed from real world translation, here is an overview of the system I want to build:

1. Choose a section of text to be translated in one session (could be a few pages or an entire chapter)
2. Read the text to see what you are up to
3. Make a list of terms non-trivial to translate (like character names, places names, specific expressions...)
4. Recall if you already decided on a translation for each of those terms in the past
5. If recall failed, try searching in the reference translation, or come-up with an original translation
6. If recall failed, make a translation decision based on the previous step
7. Think about the text for a while, ponder translation trade-offs, search the reference translation some more
8. Write a translation draft
9. Proofread with fresh eyes
10. Make adjustments based on review
11. Go back to 1.

Let's build this thing!

Disclaimer: even though the specification of each step and tool of the pipeline was made by yours truly, the python implementation was realized by Codex with GPT 5.6 Sol medium over the course of a week. (The first workable version was done in one day, the six others were used to refactor the project into a more robust, generic and efficient pipeline).

## Book pre-processing

### Making the reference translation semantically searchable

Multiple steps of the proposed pipeline are about finding how a given string of text has been translated in the reference translation. What I want is some kind of process that takes a query in natural language, and returns the closest matches in both english and japanese in the reference translation with some surrounding context.

The starting point for search is simply searching japanese terms returning the corresponding english context around that term so the LLM can identify the translation. However language is messy and sometimes you want to look for indirect matches like paraphrases "the officer keeping the fleet supplied".

This problem lead to the development of interesting solutions like `word2vec` in the pre-transformer era. The idea is to associate each token with a vector from a vector space that we hope will capture some semantic meaning. (king - man + woman = queen). 

This process is called embedding, and is a key part of modern LLMs as well: convert the input tokens into vectors, add an empty vector at the end for the output token, run the attention mechanism on all those vectors, then translate the final vector into a token using the inverse embedding.

Searching using embeddings is straightforward: transform the natural language query into a vector, and find the closest other vectors inside the list of stored embeddings.

The question is then, how do we make good english/japanese pairs to then embed?

First we will choose a size for those pairs: too small and we would lose much of the surrounding context, which would lead to suboptimal translation decisions. Too large and we would get too many matches for any query, which would drown the signal under the surrounding noise.

I chose a size of 500 japanese characters for starters. Of course it can be tweaked but remember it is a trade-off.

The next step is splitting the book into chapters as we can be sure the start of the source and target chapters are aligned.

For a given chapter, we then take the ratio of english characters per japanese character and deduce an english size for each pair.

We then split the chapters of both the japanese and english books using the language-specific chunk size to get our aligned pairs. This technique works best for chapters that are not too long, as very long chapters would lead to misalignment of the chunks, rendering our semantic search useless.

Now that we have our list of translation pairs, we will embed each pair as one vector using the [BGE-M3 model](https://huggingface.co/BAAI/bge-m3). It performs well for multilingual contexts, which is exactly what we need. Even better: it can run locally.

Now we can run both the simple search and the semantic search to get the best results possible.

For example we can query a name:

```sh
uv run book-translate reference-search "キャゼルヌ"
```

It is pronounced "Kyazerunu", so translating it to english is not obvious at all. The search tool returns passages mentionning "Caselnes" as lexical matches, which is indeed how the name was translated by Daniel Huddleston.

And because this is a semantic index, we can make more complex queries:


```sh
uv run book-translate reference-search "A man who wanted to study the past but ended up commanding on the battlefield"
```

Getting the following result:


```json
{
    "alignment_id": "ALN-VOL03-00071",
    "source_text": "「むろん、喜んでおいでよ。口にはお出しにならないけど……」\n\n彼女としてはそう答えるしかないであろう。通話を終えて、少年はすこし考えこんだ。\n\nヤンはユリアンを軍人などにしたくないのである。だが、ユリアン自身は軍人志望であり、ヤンとしては自分の意思を少年におしつけるわけにもいかず、いっぽうで自分の手もとにおいておきたい心情もあり、この件にかんしては〝同盟軍最高の智将〟も言動に整合性を欠くことはなはだしかった。\n\nなにしろヤン自身の職業選択は没理想のきわみなのだ。無料で歴史を学べる学校を探して、士官学校の戦史研究科にはいり、それが中途で廃止されたので、いやいや戦略研究科に転じ、ひとかけらの喜びもなく軍隊にはいったのである。\n\nそれに比較すれば、ユリアンの軍人志望のほうが、よほど主体的で、職業にたいしても自分自身にたいしても誠実というものだろう。ヤンがとやかく言う筋合はないはずである。はずではあるが、ユリアンは、やはりヤンにこそ彼の進路を祝福してもらいたかった。\n\nユリアンの父は軍人だったが、その死後、ヤンのもとで育たなかったら、ユリアンはかならずしも軍人志望にはならなかったろう。よしあしはべつとして、ヤンの人格的影響はユリアンに大きくおよんでおり、ヤンとしては、少年の軍人志望にとやかく言えば、鏡にむかってしかめ 面 をしてみせる結果になるのである。",
    "target_text": "It was probably the only way she could have answered him. After the call ended, Julian sank into thought for a while.\n\nYang had never wanted to turn Julian into a soldier. Julian himself, however, wanted to be a soldier. As for Yang, he didn’t feel he should force his own wishes on the boy, but at the same time he wanted to keep him close by. This was one matter in which the words and the deeds of the most brilliant admiral of the alliance had been highly inconsistent.\n\nIn any case, Yang’s own vocational choice had been an extreme case of life not following its intended script. After looking around for a school where he could study history for free, he had entered the Department of Military History at Officers’ Academy—only to have his department abolished along the way, to be transferred against his will to the Department of Military Strategy, and then to enter the military without so much as a spark of enthusiasm.\n\nIn contrast, Julian was really taking the initiative in his martial ambitions, and being true both to his chosen profession and to himself. This shouldn’t have been any of Yang’s business. It shouldn’t have been, but Julian really did want Yang’s blessing on the course he had chosen.\n\nJulian’s father had been a soldier, but if Julian had not been raised by Yang after his death, it was far from certain that he would have set his sightson the military. For good or ill, Yang’s personality had exerted a powerful influence on Julian, and if Yang were to criticize the boy’s career choice now, he would only end up scowling at himself in the mirror."
}
```

If you don't want to read the whole passage, just know it indeed references the backstory of Yang-Wen-Li even though the wording is not the same at all: `After looking around for a school where he could study history for free, he had entered the Department of Military History at Officers’ Academy—only to have his department abolished along the way, to be transferred against his will to the Department of Military Strategy`.


### Chunking into segments

Now that the reference translation is usable, we need to prepare the japanese source we actually want to translate. 

Much like before, we want to chunk the text into segments of a given length, with a tradeoff.

Too long: the LLM will need to make too many unrelated decisions to make a good translation.
Too short: the LLM might miss some long range dependencies that are needed to make a good translation. 

My first instinct is again to split by chapters first, then subdivide again if chapters are too long.

As a rough starting point, I went with a segment length of at least 2000 characters, and at most 6000 characters. For 200 pages, that's about 30 segments, so between 6/7 pages per segment.

## Incremental glossary

Going back to humanmimicry, a human translator would not need to check the reference translation every time it sees the name of the main character. Instead they would have either in their mind or written down, a glossary for reoccuring names.

Of course when the translator sees a name or expression for the first time, it is not inside the glossary, so we use our semantic search from earlier to find clues and examples in the reference translation. 

The translator can then make an informed decision that we can record in the glossary (and even include the relevant passages as evidence).

The glossary then acts as a shared memory between translation sessions. Later sessions are very likely to reuse what they find in the glossary next time they encounter the name or expression.

## Review

Good translators read their output multiple times to find mistakes that could have happened while writing the translation draft. Human attention is limited hence it needs multiple passes to reach a stable result.

LLM attention also has weaknesses, and so stopping at the first draft would be a mistake.

Given our problem, we want to review 2 aspects of the translation:
- Is it correct? (The english text is correct and conveys as much of the meaning of the original japanese)
- Does it follow the reference translation style? (A correct translation that is painful to read would not be much of an improvement to our initial setting)

Those 2 review dimensions are orthogonal, and as such they should be handled by 2 different agents, each with a prompt tailored to their task.

## Model choice

We now have a solid translation pipeline, what LLM are we going to plug inside?

We could technically chose a different LLM for each role, but that would mean finding which LLM works best for a certain task, which is time consuming. For the sake of simplicity we will use the same LLM for all agents.

After working on some coding tasks with Deepseek V4 Flash, I found it lacking in the taste/decision making area, which is critical for a good translation. And I don't want to spend thousand of dollars on the translation either, it's really a fun side-project before anything else.

I settled on Deepseek V4 Pro after testing it on some coding tasks. It is about 3x more expensive than Flash, but also has better taste than its sibling.

Taking our original estimate, that would put the task at O(30$), which is more than reasonable.

Let's start the pipeline shall-we ?

## Results

Over the course of 4 days, all 3 books have been translated! The pipeline was not running 24/7 as I was making adjustments along the way, and also my API key had no balance left for an entire night!

My estimate is about 17h per book which is 17h for 200 pages, so 8/9 hours for 100 pages.

How much did it cost?

![API cost](./finalCost.png)

In the end it cost about 20 bucks for the 3 books, what a relief! 

You might notice the token count of about 390M and think, "hold on our estimate was about 36M output tokens!". The number displayed on deepseek usage board conflates input and output tokens. Here are the output tokens day by day:

13th of july: 2.3M output tokens
14th of july: 3.7M output tokens
15th of july: 4.4M output tokens
16th of july: 2.3M output tokens

Total is 12.7M tokens, so our 100x cognitive production estimate was quite off by a factor of 3 haha! At 120k tokens per book, the pipeline used 3% of its token production for the final translation and not 1% like I guessed at first.

Notice how token usage went up everyday as I fixed some outages in the pipeline. Including a depleted API key!

Are any of those tokens any good?

But is it any good? How do you even measure something like that? Translation is a hard to verify domain after all.

I only have qualitative tests to share with you. My partner and I read side-by-side some of the hardest passages of Tyran Grillo's translation with the newly machine translated one. It wasn't a blind experiment as she already had read those passages, in fact those passages motivated the entire operation you see. So take this with a grain of salt.

We found the machine translation to be significantly easier to read, and some of the sentences that had illogical meanings were making perfect sense in our version. Fast forward a few days later, my partner feels less friction when reading the book (I don't know yet myself I am still reading the 3rd book when I write these words lol).

The only mistake she found was the translation for the character name "Caselnes", that was found to be "Cazerne". When asking Codex about it, it innocently told me the glossary mechanism I proposed was not implemented during the translation of the 4th book. Do not trust your AI agents!

Thankfully the 5th book's translation had only started and so I made sure the glossary was wired up before continuing the process. I might return to it and rerun the full pipeline again just to fix the 4th book haha.

## Conclusion and ethical considerations

Let's reflect on what we just did today. We were able to make a better translation of a japanese book in english, at an affordable cost, allowing more people to read it through the language barrier. This was made possible by using an LLM to build a pipeline around another LLM, both of them trained on countless stolen books. Furthermore, we used the work of a human translator to imitate his translation style without prior authorization. If you *should* feel conflicted about  this.

I do not want a world where human translators are unable to live decently because their honest work no longer pays their bill when competing with LLMs.

I do not want a world where we demonize using our technology to build bridges across cultures. The first step toward war is to strip your designated ennemy from their humanity, building understanding between cultures prevents that.

We should blame the unfair society that lets translators be the victims of technological progress. And they are not alone, what about artists which now have to compete with image models? What about programmers who may have to compete soon with next generation coding agents?

I won't pretend I know the solution to this problem. A Universal Basic Income (UBI) founded by some kind of AI tax could be part of a long-term solution, but people are suffering right now while the transition has not gone far enough where we could sustain that kind of UBI it seems. 

For my part, I will not be distributing copies of the new machine translation. I will not sell access to my pipeline either. But I want people to have the option to translate their own books, thus I am making the code used for this experiment freely available under a copyleft license on github [LINK].
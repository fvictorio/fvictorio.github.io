---
title: "An underqualified reading list about the transformer architecture"
date: 2025-10-30
---

When I see a list of learning resources, I assume that whoever curated it knows much more about the topic than what I’ll learn. So you wouldn’t be blamed for thinking that I have read 100x the amount of things about the transformer architecture than what I’m sharing here. In reality, that multiplier is shamefully close to 1x. Put another way: I have no idea what I’m talking about.

But I think this list can be useful anyway, because it has a couple of somewhat idiosyncratic goals: **conceptual understanding** and **historical perspective**. By conceptual understanding I just mean having a slightly better mental model of what’s going on under the hood when you use a tool like ChatGPT. And by historical perspective I mean understanding how this architecture fits in the bigger narrative of artificial intelligence research.

Having those goals means that this list is not _practical_: there’s nothing about prompt engineering or how to use an API, let alone how to implement or train an LLM, or even run one locally. There are also no philosophical musings like “well, but is this _really_ thinking?” nor, God forbid, any ethical discussions about alignment or whatever. But if you want to get a basic understanding of how transformers work and have a (short, incomplete, probably wrong) narrative of how they came about, then I’d say this post is for you.

# Pre-requisites

I’m assuming you have some understanding of how neural networks work. If the following two sentences make sense, then I’d say you can easily read everything I’m recommending in this list:

- The weights of a neural network are set during training, using training data and the backpropagation algorithm.
- A forward pass in a neural network is mainly just a bunch of matrix multiplications.

If (like me) you kind of understand them but it’s been a while since you’ve used this stuff, then the first four chapters of [3blue1brown’s series on deep learning](https://www.youtube.com/watch?v=aircAruvnKk&list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi&ab_channel=3Blue1Brown) are a good refresher. They might also work as an introduction if you are starting from scratch, but I can’t say for sure since that wasn’t the case for me.

# The list

I’m going to sort this list in the order I think makes the most sense. In addition to that, each section is grouped by a topic and driven by a question. To be honest, I didn’t read the things here in the order I’m using; it was way more chaotic. But this is how I would explain it to someone else.

## The bitter lesson

There was a comment about transformers that I kept hearing: that one reason they were revolutionary was that they were very parallelizable. This meant that they could be trained faster and therefore use more data during training. And while it makes sense that this was a big deal, I was still intrigued: how something as prosaic as parallelizability could have unleashed the current AI boom? In reality this is only part of the explanation, but still.

Turns out there is a perfect answer to that question: [The Bitter Lesson](http://www.incompleteideas.net/IncIdeas/BitterLesson.html), by Richard Sutton.

This is apparently a classic within the AI community, and I can see why. It’s powerful, packed with wisdom, persuasively argued and, on top of all that, it’s very short. Read it, re-read it, and read it again after finishing this list. It’s fantastic.

## Recurrent Neural Networks

Ok, so the transformer is more parallelizable, and that matters. But more parallelizable than _what_? The most common comparison is with Recurrent Neural Networks (RNNs), which apparently were the state of the art before transformers stole the spotlight.

Do you need to understand RNNs to understand transformers? Not at all. But I’m including this section anyway for two reasons. One is that they help with the historical perspective, which is one of the goals of this list. But also because the two resources I have to share are excellent. They were written by people who really know what they are talking about, and who love teaching.

The first is [Understanding LSTMs](https://colah.github.io/posts/2015-08-Understanding-LSTMs/), by Chris Olah[^1]. If you are not that interested in the topic, you can read just the first two sections (“Recurrent Neural Networks” and “The Problem of Long-Term Dependencies”), and the Conclusion at the end. But I’d still recommend reading the whole thing, because the explanation is great and LSTMs (a type of RNN) are quite interesting.

The second RNN-related resource is [The Unreasonable Effectiveness of Recurrent Neural Networks](https://karpathy.github.io/2015/05/21/rnn-effectiveness/), by Andrej Karpathy[^2]. As with the previous article, you can focus on the beginning and the conclusion, but all of it is worth reading.

## Attention

The two articles about RNNs I recommended above make similar remarks in their closing sections (emphasis mine):

Colah:

> LSTMs were a big step in what we can accomplish with RNNs. It’s natural to wonder: is there another big step? A common opinion among researchers is: “Yes! There is a next step and it’s **attention**!"

Karpathy:

> The concept of **attention** is the most interesting recent architectural innovation in neural networks.

What is this curiously named concept?

One good first explanation is [Mechanics of Seq2seq Models With Attention](https://jalammar.github.io/visualizing-neural-machine-translation-mechanics-of-seq2seq-models-with-attention/), by Jay Alammar. It explains attention in the context of RNNs, which makes the article a bit dated. But at the same time, that’s a good thing, because it fits with the simplified narrative we are building.

There’s much more to say about the attention mechanism, but it’s time to jump to what we are really interested in.

## Transformers

I had heard the terms “attention” and “transformers” together a lot. I had also heard about the legendary paper “Attention is all you need”. In my fuzzy understanding, I coupled these two concepts, thinking that transformers and attention were kind of synonyms. But as we saw in the previous sections, that’s not the case, and people were talking about attention years before this paper was published.

Besides, the title of the paper is very self-descriptive! It’s not called “Introducing the attention mechanism”. The main idea of the paper is that you can do a lot with _only_ that mechanism, implying that it already existed.[^3]

So we have a basic understanding of what the attention mechanism is, and the vague idea that a “transformer” is an architecture that relies a lot on that mechanism. A good next step is [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/), also by Jay Alammar. As with the attention explainer, this one is outdated (more on this soon). But, again, this is helpful to understand how things developed historically. Even if the encoder-decoder architecture explained in the article is (I’ve been told) not used anymore, it shows how transformers evolved from the “sequence-to-sequence with RNNs” world, which is nice.

If after reading The Illustrated Transformer you feel like saying “great, I understand transformers now, I can stop here and go brag about it”, then that’s it, at least for this section. But I personally think that deep topics are best understood through multiple perspectives, so I’ll share two resources more.

One is the [rest of the 3blue1brown series on Deep Learning](https://www.youtube.com/watch?v=wjZofJX0v4M&list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi&index=6). As I mentioned in the [Pre-requisites](#pre-requisites) section, the first four chapters are good for learning/reviewing classical neural networks; the rest of the series focuses on transformers and LLMs.

<details>
<summary><b>Aside</b>: A brief comment about learning from videos</summary>

<p style="margin-left:2em;">I have to say I'm a bit skeptical of video explanations, unless you have the discipline to watch them in a very active way: pausing to think, taking notes, re-watching parts you didn't fully understand. But I personally find that the medium is not conducive to that. Your experience can be different, of course, but I myself only use videos to get things explained in a different way, or when I'm only interested in getting a quick, shallow understanding.</p>
</details>

The other resource happens to be my favorite article in this list: [Transformers from scratch](https://peterbloem.nl/blog/transformers), by Peter Bloem. This is a more rigorous and modern explanation, but that’s not the reason I like it so much. It’s because it approaches things in the way I like to teach and learn: start with the essence of things, and gradually add complexity as needed. Take key/query/value vectors, for example. Other explainers will mention them in a way that feels almost arbitrary. Here, on the other hand, they are just additional tricks on top of the core idea (quick question: why are there three kinds of these vectors? Why not four? While the article doesn’t spell it out, you’ll be able to answer this anyway). I have to admit that I only skimmed the code sections, even if I agree in principle with the idea that “What I cannot create, I do not understand”. It just wasn’t part of my learning goal.

## Large Language Models

While up to this point we’ve only talked about neural networks and attention mechanisms and transformers, let’s not pretend we aren’t learning these things because of our new hallucinating overlords. But there’s a reason I haven’t mentioned LLMs so far. Just as it’s easy to conflate transformers with the attention mechanism, even if they are conceptually separate things, it’s also easy to think that transformers and LLMs are somehow equivalent. And that’s not the case, in either direction. An LLM doesn’t _have_ to use the transformer architecture (although all of the big ones do today). To see how this is the case, watch [The 35-Year History of LLMs](https://www.youtube.com/watch?v=OFS90-FX6pg)[^4].

Even more importantly, the transformer architecture can be used for many other things. Quoting the definition of transformers from “Transformers from Scratch”:

> Any architecture designed to process a connected set of units—such as the tokens in a sequence or the pixels in an image—where the only interaction between units is through self-attention.

In other words, if your input is “a connected set of units” (a text, an image, a non-linear logographic alien language), then using transformers is a good bet.

# Closing words

What’s next? I don’t know! As I said in the introduction, I haven’t read much more on the topic than what I shared. But, true to the spirit of talking without knowing, here are some possible next steps:

- The [Attention is all you need](https://arxiv.org/abs/1706.03762) paper. Obviously outdated, but just skimming it is interesting for historical reasons. There’s a mix of the authors knowing that this is really cool with the fact that they obviously couldn’t predict what they were unleashing. It’s also interesting how focused it is on language translation, given the much broader applicability that transformers have shown.
- [Let’s build GPT: from scratch, in code, spelled out](https://www.youtube.com/watch?v=kCc8FmEb1nY), by Andrej Karpathy. I admit I haven’t watched this one, but I’ve seen it mentioned multiple times as a great resource.
- [Developing an LLM: Building, Training, Finetuning](https://www.youtube.com/watch?v=kPGTx4wcm_w). A video about the real-world details of building an LLM. I think this is the least well-known resource in this list, and the most advanced, but I think it’s a good complement to the rest of the material given that it’s very grounded in reality.
- Go and re-read [The Bitter Lesson](http://www.incompleteideas.net/IncIdeas/BitterLesson.html) again.

---

[^1]: One of the founders of Anthropic. He also worked at OpenAI and Google Brain.
[^2]: Famous [speedcubing influencer](http://badmephisto.com/). Also a founding member of OpenAI and former Director of AI at Tesla.
[^3]: This reminds me of a passage in the classic [_How to read a book_](https://en.wikipedia.org/wiki/How_to_Read_a_Book) about how we don’t pay enough attention to titles. The authors say this with respect to Gibbon’s _The Decline and Fall of the Roman Empire_: «Nevertheless, when we asked the same twenty-five well-read people why the first chapter is called “The Extent and Military Force of the Empire in the Age of the Antonines," they had no idea. [...] Unconsciously, they had translated “decline and fall” into “rise and fall." They were puzzled because there was no discussion of the Roman Republic, which ended a century and a half before the Age of the Antonines. If they had read the title carefully they could have assumed that the Age of the Antonines was the high point of the Empire, even if they had not known it before. Reading the title, in other words, could have given them essential information about the book before they started to read it.»
[^4]: Apparently the title of the video has been changed to “The Origin of ChatGPT”, for what I guess are obvious reasons.

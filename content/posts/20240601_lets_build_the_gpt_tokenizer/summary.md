---
title: "[Lecture notes] Let's build the GPT Tokenizer" 
date: 2024-06-01
tags: 
- LLM
- Tokenization
draft: true 
---

Andrej Karpathy has released a great series of in-depth-hands-on of building GPT models. Here's my notes taken during watching the videos.


Tokens are the atoms of a Large Language Models (LLM)
Tokenization is the process of translating text into sequence of tokens.
Many issues of LLM are mainly due to tokenization:
* LLM are bad at simple arithmetic 
* LLM doesn't preform well on non english languages. 
* LLM can't do simple string manipulations like reversing a string

An example of tokenizing text to words is shown below, each token is shown in a different color. Notice spaces are part of the tokens!
Some weirds phenomena: 
* "127" is a single token while "677" is represented by two tokens. 
* " Egg" and "Egg" are represented very differently. 
* Every space in Python is considered as a single token. When having a long of indentation all the context length is overflowed.
* For non-english languages, sentences are represented by much more tokens. This creates an issue with the context length of the LLMs.  
![Tokenizer](/posts/20240601_lets_build_the_gpt_tokenizer/tokenizer_example.png)


There's a sweet spot of the number of token in the vocabulary
Increasing the number of tokens in vocabulary makes a sentence to be encoded to less tokens. However, this makes the embeddings table larger, and the LLM Softmax function to take a larger vector.

# Creating a tokenizer
Each character is represented in Unicode. There are several encoding from Unicode to byte-string, the most common one is UTF-8.
![utf8](/posts/20240601_lets_build_the_gpt_tokenizer/string_to_utf8.png)
Using UTF8 naively means we could have only 256 tokens (byte size) (Koby: isn't it 256*4=1024 tokens?), which is too small and would create a very long context length per input sentence. 

The solution is "Byte pairing encoding" 

# Byte Pair Encoding
We are looking for the most frequent pair of tokens and replace it with a new token. 
We repeat this iteratively until a stopping condition is met. 

## Resource
[Arxiv](https://www.youtube.com/watch?v=zduSFxRajkE)


---
title: "[Lecture notes] Let's build the GPT Tokenizer" 
date: 2024-06-08
tags: 
- LLM
- Tokenization
draft: false 
---

Andrej Karpathy has released a great series of in-depth-hands-on of building GPT models. Here are my notes taken during watching the ["Let's build the GPT Tokenizer" video](https://www.youtube.com/watch?v=zduSFxRajkE).

# What are Tokens?
Large Language Models (LLM) don't process the raw text directly. They use tokens are the out of the Tokenization process which translates text into sequence of tokens.
Many issues of LLMs are mainly due to tokenization:
* LLMs are bad at simple arithmetic.
* LLMs perform poorly with non-English languages. 
* LLM can't do simple string manipulations like reversing a string.

# Example of Tokenization
An example of tokenizing text to words is shown below, each token is shown in a different color. Notice spaces are part of the tokens!

Some weirds phenomena of the Tokenization process: 
* The number "127" is tokenized as a single token, whereas "677" is split into two tokens.
* The tokens for " Egg" (with a space) and "Egg" (without a space) are different.
* In Python, every space is treated as a separate token, which can cause the context length to overflow if there are many spaces.
* Non-English sentences are tokenized into more tokens, reducing the effective context length the LLM can handle. 
![Tokenizer](/posts/20240608_lets_build_the_gpt_tokenizer/tokenizer_example.png)


# Creating a tokenizer
Each character is represented in Unicode. There are several encoding from Unicode to byte-string, the most common one is UTF-8.
![utf8](/posts/20240608_lets_build_the_gpt_tokenizer/string_to_utf8.png)
Using UTF8 naively means we could have only 256 tokens (the maximum number a byte represents), which is too small and would create a very long context length per input sentence. 

The solution is "Byte Pair encoding" 

## Byte Pair Encoding
We are looking for the most frequent pair of tokens and replace it with a new token. 
We repeat this iteratively until a stopping condition is met. 

An important measurement for the tokenizer is the compression ratio: the text length in characters / the text length in tokens

# Regex patterns
We don't want to map 'dog.', 'dog!', 'dog?' to a single token since the semantic meaning is different. 
In gpt-2 code, there are additional merging rules to prevent such merges implemented as [regex](https://github.com/openai/gpt-2/blob/master/src/encoder.py#L53). This regex enforces rules to prevent merging of token.

Some of the rules are:
1. Not merging sequences with space between.
2. Numbers cannot be merged with letters.
3. The character ' split tokens.
4. White spacing are included in the beginning of the words (" are").
5. Punctuation are grouped together 
The implementation splits the input string to a list by the regex. Then apply byte pair encoding such that different elements in the list cannot be merged.
![regex patterns](/posts/20240608_lets_build_the_gpt_tokenizer/regex_rules.png)

# Special tokens
there are 256 raw byte token. GPT does 50,000 merges. Therefore we should have 50256 token in the dictionary.
In addition, there's an additional 1 special token: '<|endoftext|>'.
It's used to delimited documents in the training set. This hint the language model it should ignore the previous tokens before the <|endoftext|> token when it produces text.
There's additional tokens to distinguish between user and chat answers. 

It's very common to train an LLM with special tokens. When adding a new token we should keep in mind
1. Add an additional row to the embedding matrix (in which each row stand for a specific token) that is later fed to the LLM.
2. Extend the final layer of the LLM to be able to predict the new token
Base model -> chat model.

## More information
* In deep learning, We want to keep the raw data as much as possible, no text normalization
* Characters that were not in the training set have a fallback to utf8 byte. Without bytefallback we get the 'unk' token. We better to have bytefallback true. We better feed the model with different tokens not just a major chunk of known.
* LAMA2 uses the tokenizer that adds space (' ') to every beginning of sentence such that ' hello' and 'hello' are mapped to the same token. 
* Having space in the end of the sentnce you want to model to complete makes the complition much worse since ' ' is encodedhas the token but in the wild most complition are with ' word'


# Set vocabulary size

There's a sweet spot of the number of token in the vocabulary
Increasing the number of tokens in vocabulary makes a sentence to be encoded to less tokens. However, this makes the embeddings table larger, and the LLM SoftMax function to take a larger vector.


Vocabulary size affects the token embedding table: number of rows. Each token is associate with a raw that is trained with backpropogation. 
The last layer of the LLM also depends on the vocabulary size since we want to create a probablity assignment for each token in the vocabulary. When having more tokens, we produce more probablities. 

Having a large vocabulary:
1. Each token becomes more rare during the training process, vectors becomes under-trained.
2. Text are squicshed togehter.
3. More computation.

Positive
2. We can feed the model larger texts.


When doing finetuning, many special tokens are introduces to provide more context to the model: broswer interaction, chat respocne etc.
Training only the new parameters and freezing the rest.

# Multimodality 
Recent trasformers can take both images and text and produce either image and text.
Man approaches take image, chunk it to integers and use it as tokens alonng with text tokens.

Sora took visual encoder to create token patches. Then use it as tokens.   

![multimodal](/posts/20240608_lets_build_the_gpt_tokenizer/multimodal.png)


## Resource
[Youtube video](https://www.youtube.com/watch?v=zduSFxRajkE)


[sentencepiece:](
https://github.com/google/sentencepiece) A common tokenization library. Was used in Llama2. Not very recommended by Andrej, too nuance settings.

tiktoken : GPT4 tokens. Doesn't have training code.


minmbpe :Andrej libaray to train,encode, and decode.
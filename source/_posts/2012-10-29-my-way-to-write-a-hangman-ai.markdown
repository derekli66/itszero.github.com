---
layout: post
title: "(my way to) Write a Hangman AI"
date: 2012-10-29 20:17
comments: true
categories: AI coding hangman
---

I was asked to write a hangman AI as a challenge last week. I was asked not to
leak the detail, so I will not post my solution here. However, the hangman
problem itself is a well-known problem, I would like to share some thoughts
that I used to build my hangman solver.

Preface
-------

There is a [hangman] page provides detailed information for hangman
game. However, I would like to describe it in a shorter, more programmer's way.

### Input

You will receive an serialized object containing following information:

* state: The state is a string that reveals your correctly guessed characters
  in the string. For those unrevealed characters, a ``_`` is placed in those
  positions. For example, say the answer is "apple" and you guessed *p*, the
  state string will be ``_pp__``. Please note that the state could contain
  multiple words, a sentence.

* remaining_guesses: How many times you can make a wrong guess before the game
  is over.

* status: It could be one of {__WIN__, __ONGOING__, __LOSE__}. The game will
  start as __ONGOING__.  If you reveal all characters before you used up all
  guesses, you __WIN__ the game. If you make too many wrong guesses, you
  __LOSE__ the game.

### Output

For each game, you will start with a empty state. You must keep giving out a
character as a guess until the status is __WIN__ or __LOSE__.

Thoughts
--------

Before we talk about anything, I need to clarify one thing: I am not familiar
with NLP(Natural Language Processing) techniques. I might use some dumb way to
solve this problem, but, well, it's at least something that my intuition leads
me to.

### Initial Guess

If the state is all unrevealed, I will do an initial guess. The initial guess
uses vowels, the famous a-e-i-o-u. However, I use a little variation here. I do
not guess the vowels in this order, I sort it by the frequency of characters
appearing in dictionary. I guess by e-i-o-u-a. The initial guess stops whenever
a character works, then we go to word attack mode.

### Word Attack

Since I don't have any knowledge in the language model, I will launch a
word-based attack. In my implementation, I only attack one word at a time. It
is very important that we choose the right word to attack. If we pick the wrong
word to guess, the probability to produce a correct guess will decrease. First,
I assume that, in most cases, the word-to-be-attacked will be included in my
dictionary. If we don't make this assumption, it is hard to do optimization
since you have to assume you have no knowledge regarding the context.

#### Which Word?

The best word to attack, in my opinion, is the word having most characters
solved and having longest length as possible. Now, to decide it, the formula I
used is:

{% raw %}
<div class="mathjax">
$$
f_{factor}(x) = \frac{f_{unsolved}(x)}{f_{strlen}(x)^2}
$$
</div>
{% endraw %}

The best word to attack will have smallest factor from this formula. It first
calculate the percentage of unsolved characters in the word, so the word with
most solved characters now having the smallest factor now. However, say we have
a word ``a_``, the word is now 50% solved but to finish the second character,
you only need one move. It sounds good, but it does not generate any
*side-effect*. Say now we have a word ``co__ec__e__`` and you already know the
full word is ``correctness``. You can now confidently send answers ``[r, t, n,
s]``. You may reveal characters in other words in the process, thus increase
the overall probability of making right guesses.

#### The n-grams

{% blockquote Wikipedia http://en.wikipedia.org/wiki/N-gram N-gram %}
In the fields of computational linguistics and probability, an n-gram is a
contiguous sequence of n items from a given sequence of text or speech. An
n-gram could be any combination of letters. However, the items in question can
be phonemes, syllables, letters, words or base pairs according to the
application. The n-grams typically are collected from a text or speech corpus.
{% endblockquote %}

Since I'm not a NLP guy, I will just quote the definition of n-gram from
Wikipedia. In my solver, I use two type of n-grams: A bigram table and a
unigram table. Before I calculate those, I will filter the dictionary to a
smaller version by eliminating words using the current state and guessed words.
For example, the state of the word I'm attacking is ``a__le``, and I haven't
guessed ``[p, z]``. I will generate a regular expression of ``a[pz][pz]le`` and
apply it on my dictionary. Then, I calculate those n-grams table in current
reduced context.

##### Unigram table

My usage for unigram is very simple. I merely calculate the frequency of a
characters shows in my dictionary.

##### Bigram table

The bigram I'm trying to build is based on the connection for two continuous
characters. I will split each word into bigrams using the following ruby code:

```ruby
def bigram_split(str)
  "^#{str}$".split('').each_cons(2).to_a
end

>> bigram_split("hello")
=> [["^", "h"], ["h", "e"], ["e", "l"], ["l", "l"], ["l", "o"], ["o", "$"]]
```

I collect those bigrams and calculate the frequency for each of those showing
up among whole bigrams space for current dictionary.

#### The Attack

Now everything that we need before launching the attack is prepared, we can
start to work on a guess. The first step is to acquire bigrams from the word
state. The only bigrams we need is those with one of the character unsolved,
but not all unsolved. In a more direct way, what we need is ``[*, '_'] ['_',
*]`` but not ``['_', '_']``. For those unsolved bigrams, I use the bigram table
I just calculated to get a count on how frequent of a character shows up after
or before a known character. I then sum the count for all bigrams in the word
state and pick the character with highest count as the guess. It's that simple.

#### Falling back

Well, sometimes you just don't have any clue. In that case, my program will
first falls back to attack on second best word and so on. If all words are
tried, but we still have no clue. I'll just fire a random guess using unigram
table. However, it is really a wild guess, the success rate of making a correct
guess using unigram table is not high.

Future Work
-----------

There are many ideas that can be implemented to improve my solution. Like
implementing a word bigrams to further weight the possibility of characters
bigrams could be useful, but it would require more research into NLP field. I
think it is really a fun challenge. If you got some spare time, try it!

[hangman]: http://en.wikipedia.org/wiki/Hangman_(game)
[n-gram]: http://en.wikipedia.org/wiki/N-gram


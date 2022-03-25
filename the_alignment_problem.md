# The Alignment Problem

## Prologue

Walter Pitts has a Never-Ending Story night in the local library, sparking his love of logic.

Jerry Lettvin and Walter meet at Bertrand Russell's lecture hall. Lettvin is in medical school. Eventually both young men are taken in by Warren McCullough, a neurologist and one of Lettvin's mentors.

Thus began their work on the concept of a "neural network"; treating neurons like various logical switches. The paper they wrote (1943), "A Logical Calculus of Ideas Immanent in Nervous Activity", wasn't very impactful on the field of biological neuroscience, but it gave birth to an entirely new field.

## Introduction

### `word2vec`

It's 2013. Google publishes its open-source project "word2vec", made by feeding loads of natural language to a biologically-inspired neural network. The representation of words as vectors made it possible to do math with the words, `China + river = Yangtze`, etc. The program was put to use in many areas and it was 2 years before problems were discovered.

It's 2015. During a happy hour at Microsoft Tolga Bolukbasi and Adam Kalai were playing with `word2vec` and got surprisingly sexist responses.

COMPAS, used to gauge risk of recitivism in parolees, is racist.

Dario Amodei is finding that gaming AI cuts surprising corners if not explicitly prevented from doing so.

How do we get AI to do what we want, when we want, and not what we don't want- particlularly when this is difficult to convey.

2 areas of concern: present-day AI and the future of AI.

## Prophecy

### 1. Representation

#### Perceptron/ Rosenblatt

Greeted with excitement though it has "no real use", a shallow neural network.

Stochastic gradient descent

Book, Perceptrons, the limitations of the perceptron.

Everyone mentioned earlier dies.

#### Alexnet

Deep neural networks are being used in the late 80s and 90s but are limited by the speed of computers and training data.

Amazon's Mechanical Turk (!!!) saves the day woth ImageNet, a large dataset of images assembled and labeled. Artificial artificial intelligence.

2000s - GPUs change everything, particularly with regard to training, because they can process so much in parallel.

Alex Krizhevsky, a deep learning researcher, wins an ImageNet contest wiht AlexNet.

#### The problem

Google Photos image recognition labels photos of people with darker skin tones "gorillas". This is only fixed to this day by removing the "gorillas" label altogether.

It's not a biased model/ algorithm. Its the training data.

- camera tech is biased
- so is fillm technology
- Shirley card

#### Fixes

Joy Buolamwini, in the early 2010s, using a neural network to recognize faces.But it wouldn't recognize hers.

Training data, like IMageNet or LFW, ends up having fewer representations of minorities both in terms of race/ gender and lighting, quality.

Parliamentarian data set.

#### Back to `word2vec`

Predictive models for missing words, computational linguistics.

"curse of dimensionality"

The solution: distributed representaitons. (e.g. `word2vec`) each word's representation is a measurment of how similar it is to any other. Using a model trained on stochastic gradient descent, we can predict the next nearest word.

But it's sort of too good. It distills information out ot the relationships between words that we'd rather not see.

#### Not just Demonstration but Perpetuation

- Resume relevance tools using such models prefer males.
  - Name
  - certain male keywords
  - extracurriculars





---
title: "ARC: Abstract Reasoning Corpus"
last_modified_at: 2025-03-09T16:20:02-05:00
categories:
  - Blog
  - AI
tags:
  - ARC
  - Abstraction
  - Emergance
---

What is so hard about tasks like ARC? Why could transformers fail to count or add? Why does their hallucinations show such a grave failure in understanding that it puts into question how they output such reasonable responses in adjacent situations?

Well, let's start with looking deeper into what we have built. If we built something so much different than humans, why is it different? And how is it the same?

These are slightly easier questions to start with. We see differnences in confidence in certain beliefs, there is no world model in the sense that an llm has an underlying concept, trys to discover the implications of that concept and plan out the future. We can see that, through natural language, it is attempting to do this, however, it does fail to organize these thoughts. But this should come as no suprise. The training process was textual complettion and therefore we engineered the machine not to understand but to predict... this seems to be small while I type it but in reality it is a giant cause of ineffiency. We don't predict words, but assemble concepts, to predict the worlds and then map those concepts to languge. If we can solve ARC effiently, then we must be able to do it without using these llms as a base. We have to use nlp machines for what they were made for, turning concepts into language. 

So, how can we start this. Well we have to enter a space of concepts, a realm that respects: composition, dichotomy, causality, reason, logic. We can give this realm basic axioms that combine to create a basis that the world can live inside. For ARC, we can shrink the world to the 3 dimensions of the dataset: 2 of space and one of color. 

Inside these 3 dimensions, it seems that limitless possiblities still reign. At first glance, it almost seems that every test is a different concept. That even the examples inside of each test are different examples. 

We can address how we can build our realm so that us humans can more easily traverse it after we talk a bit about the more strict points of the project. 

Firs, in order to create a system that solves these tasks the way humans do, the system needs:
 - A way to hypothesize (traversing the concept space)
 - A way to test it's hypothsis and update it's world model (this could be test time training but it would need to be very robust. Instead it makes more sense to have a hand model that is very easiy to update and a head model that is there to handle unchanging concepts (like mathmatics and logic) this could be different models or differnt parts of the same model. Either way, the hand model will be highy sensitive to outside forces and the head model will be very inflexible)
 - A way to think a multiple levels of abstraction


First, we can observe that, just like us, these systems are inhernently chaotic. There exist small perterbations in the input space that shift the output a great deal. Do these naturally fall in a sort of predictable 

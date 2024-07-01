# Poker AI and Machine Learning Insights

## Overview

Works in progress that leverage machine learning and AI to derive advanced insights about poker strategy

### Skills
- Machine learning (scikit-learn)
- Deep learning (Keras/TensorFlow)
- Data analysis (Pandas, NumPy)
- Programming (Python)

### Background

This project builds on my [Poker Hand History Parser and Database](parser). I am currently working on a set of projects that use machine learning and deep learning to improve my own poker play (and my opponents', if they are so inclined!). 

### Ethical Principle #1: My Own Play First

Ethically, since I play poker almost exclusively against friends, I want to take steps to ensure that I don't build an unfair advantage for myself with AI-driven insights. To level the playing field, whenever possible I will train the machine learning and AI models on *my own play first* and make these insights available to the rest of the group. This is like handing all of my opponents cheat codes to beating me, specifically!

Moreover, the AI-driven insights about my own play will always be more thorough than insights about other players, simply because of the asymmetry of available data. I have access to my private hole cards for every hand I've played, whereas the hand histories don't show other players' private hole cards except in specific circumstances, such after winning at showdown or when players choose to show their cards voluntarily. Other players' hole card data will always be incomplete, and incomplete in a way that is inevitably biased towards certain kinds of hands. For example, weak hands that are folded (by far the most frequent action in Texas hold 'em) are rarely shown, and thus bias the data set, no matter how many years of data I have for an opponent.

### Ethical Principle #2: Never While Playing

These AI-driven models should only be used as study tools outside the game, *never while playing*. Using outside assistance during play (whether AI, human, or otherwise) is strictly against the rules of poker and endangers the fundamental existence of the game. As someone who greatly enjoys the game of poker, I feel very strongly about this point!

## Project Description: Hole Card Predictor

In Texas hold 'em poker, the public community cards are face up on the table and known to all, whereas each player has two private hole cards known only to them. Knowing, or at least being able to narrow down, an opponent's two hole cards is very valuable information. Can a machine learning model predict the range of hole cards that an opponent is holding, given the action in the hand so far?

We can visualize a player's two hole cards in a 13x13 grid, where the value of each card, (A)ce down to 2, is an axis. Here is the distribution of possible hole cards a player can be dealt:

![Frequency of Hole Card Combinations](images/ml/Frequency_of_Hole_Card_Combinations.png)

Pocket pairs (hole cards of equal rank) are along the diagonal, suited cards (cards where the suits match) are above the diagonal, and unsuited cards (cards where the suits do not mach) are below the diagonal. There are 13\*13=169 possible hands, but not every hand is equally likely. Unsuited cards are the most common, followed by pocket pairs, followed by suited cards, which is why the grid is darkest below the diagonal. (The scale used in this grid is the same scale used later, which is why there are no hands at the high end of the scale in this particular visualization.)




Work in progress... Will be updated as the projects mature!

One of the best ways I can level the playing field is to create a complete poker-playing bot (in a simulated poker-playing environment) that plays exactly like me, and allow my friends to play against this Greg-bot as much as they'd like to practice their real-life strategies against me. 
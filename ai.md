# Poker AI and Machine Learning Insights

## Overview

A set of projects that leverage machine learning and AI to derive advanced insights about poker strategy

<img src="images/ml-range.png" alt="Poker AI and machine learning preview" width="60%">

*This image is just a teaser. Don't worry, there will be a clickable version later!*

### Skills
- Machine learning (scikit-learn)
- Deep learning (Keras/TensorFlow)
- Data analysis (Pandas)
- Statistical and numerical analysis (NumPy, Bayesian statistics)
- Programming (Python)

### Background

Building on my [Poker Hand History Parser and Database](parser), I am currently working on a set of projects that use machine learning and deep learning to improve my poker play (and my opponents', if they are so inclined!). 

### Ethical Principle #1: My Own Play First

Ethically, since I play poker almost exclusively against friends, I want to take steps to ensure that I don't build an unfair advantage for myself with AI-driven insights. To level the playing field, whenever possible I will train the machine learning and AI models on *my own play first* and make these insights available to the rest of the group. This is like handing all of my opponents cheat codes to beating me, specifically!

### Ethical Principle #2: Never While Playing

These AI-driven models should only be used as study tools outside the game, *never while playing*. Using assistance from an agent (whether machine or human) during play is strictly against the rules of poker and endangers the fundamental existence of the game. As someone who greatly enjoys the game of poker, I feel very strongly about this point!

## Project Description: Hole Card Predictor

In Texas hold 'em poker, public community cards are face up on the table and known to all, plus each player has two private hole cards known only to them. Knowing, or at least being able to narrow down, an opponent's two hole cards is very valuable information that informs your optimal strategy in the hand. Can a machine learning model predict the range of hole cards that an opponent is holding, given the action in the hand so far?

We can visualize a player's two hole cards in a 13x13 grid, where the value of each card, (A)ce down to 2, is an axis. Here is the distribution of possible hole card hands a player can be dealt:

[![Frequency of Hole Card Combinations](images/ml/Frequency_of_Hole_Card_Combinations.png)](images/ml/Frequency_of_Hole_Card_Combinations.png)

Pocket pairs (hole cards of equal rank) are along the diagonal, suited cards (cards where the suits match) are above the diagonal, and unsuited cards (cards where the suits do not mach) are below the diagonal. There are 13\*13=169 possible hands, but not every hand is equally likely. The grid is darkest below the diagonal because unsuited hands are the most common (~1% each), followed by pocket pairs (~0.5% each), followed by suited hands (~0.3% each). Note that the color scale used in this grid is the same scale used later, which is why there are no hands at the high end of the scale in this particular image.

The above grid would be our naive guess if we knew nothing about an opponent's hole cards. But let's see if we can do better.

### Example Game State

All of the models discussed below will use the same example player action and game state, "raise first in from the cutoff." This means:

- Preflop (only private hole cards have been dealt; no community cards yet)
- Everyone has folded to the player in the cutoff (one to the right of the dealer button)
- The cutoff raises

The question is, what range of hole cards does the cutoff have when raising first in?

### Mapping Actions to Hole Cards (Act-to-HC Model)

Using scikit-learn and data from the hand history database, I trained a series of machine learning models to learn the mapping between a player's action and a given game state to the player's hole cards (action + game_state --> hole_cards). As shorthand, I refer to this kind of model as actions to hole cards, or "Act-to-HC":

<a href="images/ml/act-to-hc_model.png"><img src="images/ml/act-to-hc_model.png" alt="Act-to-HC Model" width="60%"></a>

Once trained, an Act-to-HC model can then be used to directly predict a player's range of hole cards given an observed player action and game state:

[![Act-to-HC Workflow](images/ml/act-to-hc_workflow.png)](images/ml/act-to-hc_workflow.png)

Consistent with ethical principle #1 (my own play first), here is the output of the random forest Act-to-HC model that predicts my range of hole cards when raising first in from the cutoff:

[![Absolute Hole Card Frequency for Act-to-HC Model](images/ml/Absolute_Hole_Card_Frequency_for_Act-to-HC_Model.png)](images/ml/Absolute_Hole_Card_Frequency_for_Act-to-HC_Model.png)

The Act-to-HC model is a success! These predicted frequencies very closely match the actual empirical frequencies from my own hand histories (we'll return to this point in a minute).

This range of hands is much more specific than a naive guess. Hands with an ace, especially an ace paired with another large card, are very often raised. Low, disconnected, and/or unsuited hands are very rarely raised. The grid is darkest below the diagonal because unsuited hands are more frequent than pocket pairs and suited hands.

Did you notice the seemingly anomalous 72s and 72o standing out from the rest? 72 is known for being the worst hand in poker, so some players in my poker group -- me included! -- sometimes try to win with it as a matter of entertainment/pride. And the random forest model picked up on that!

The Act-to-HC model is a success... but not very useful. 

A closer look at the grid reveals some discontinuities in the predictions, such as KK being notably less frequent than the adjacent AA and QQ:

<img src="images/ml/kk_zoom.png" alt="Zoomed in portion of Act-to-HC grid showing lower frequency of KK" width="60%">

There is no reason for this; I will always raise AA, KK, and QQ first in from the cutoff, and as these pocket pairs are equally likely to be dealt, a robust model should predict these hands with equal probability. This is actually not an issue with the model, but with the data. The empirically observed frequencies look just like this, which means that I was just dealt KK less often in this game state due to random variance. Even knowing the hole cards for every hand I've played (about 50,000 hands per year since 2021) doesn't provide enough data to smooth out these variations.

And that is the death knell for the Act-to-HC model for any data other than my own.

I have full knowledge of my private hole cards, whereas the hand histories don't show other players' private hole cards except in specific circumstances, such after winning at showdown or when players choose to show their cards voluntarily, which both limits and biases their set of known hands. If my treasure trove of known hands isn't enough to smooth out variance, the model is doomed when tasked with predicting hole cards from data that is significantly more limited and biased.

Using a model to predict hole cards requires thinking about the problem in a different way.

### Mapping Hole Cards to Actions (HC-to-Act Model)

Instead of training a model to directly predict a player's hole cards, what if we instead trained a model to play like a particular player? I trained a series of models to learn the mappings between a player's hole cards and a given game state to the player's action (hole_cards + game_state --> action). As shorthand, I refer to this kind of model as hole cards to actions, or "HC-to-Act":

<a href="images/ml/hc-to-act_model.png"><img src="images/ml/hc-to-act_model.png" alt="HC-to-Act Model" width="60%"></a>

What we really want to know, though, is a player's hole cards, which the HC-to-Act model takes as input, not provides as output. Here's where this gets interesting. If we provide the HC-to-Act model with all possible hole card hands and a given game state, it will predict the player's actions in that game state for every hole card hand they could be holding:

[![Act-to-HC Workflow, Part 1](images/ml/hc-to-act_workflow1.png)](images/ml/hc-to-act_workflow1.png)

What we have now is a player's complete strategy for that game state! When presented with an observed player action, we can ask how consistent that action is with the player's strategy for each possible hole card hand. That gives us the player's predicted hole cards, which was our goal all along:

<a href="images/ml/hc-to-act_workflow2.png"><img src="images/ml/hc-to-act_workflow2.png" alt="HC-to-Act Workflow, Part 2" width="60%"></a>

We've essentially "reverse engineered" a HC-to-Act model to predict the player's hole cards for an observed action and game state. Here is the output of the modified random forest HC-to-Act model that predicts my range of hole cards when raising first in from the cutoff:

[![Absolute Hole Card Frequency for HC-to-Act Model](images/ml/Absolute_Hole_Card_Frequency_for_HC-to-Act_Model.png)](Absolute_Hole_Card_Frequency_for_HC-to-Act_Model.png)

Notice how much smoother this probability distribution is than the earlier Act-to-HC model (here's a link to <a href="images/ml/absolute_comparison" target="_blank">view them together</a>). For example, KK is no longer notably different than the adjacent AA and QQ. This modified HC-to-Act model provides a more robust prediction that is much less sensitive to the random variance of hole cards being dealt more or less often in particular game states.

To see why this is the case, here is a different visualization that is closer to what the HC-to-Act model outputs natively. This is the same HC-to-Act model prediction as above, but on a different scale. Instead of an absolute probability distribution that sums to one across all hole card hands, the following grid shows how often *each hole card hand* takes the observed action in the given game state. Red cells are close to 100%, which means that hand very often takes the observed action, whereas blue cells are close to 0%, which means that hand very rarely takes the observed action:

[![Relative Hole Card Frequency for HC-to-Act Model](images/ml/Relative_Hole_Card_Frequency_for_HC-to-Act_Model.png)](Relative_Hole_Card_Frequency_for_HC-to-Act_Model.png)

This blue-red "relative" visualization shows that I nearly always raise AA, KK, QQ, and many other hands first in from the cutoff. This is the core of why the HC-to-Act model is so much more robust. After a reasonable minimum number of hands, it doesn't matter if I've been dealt KK five times or five thousand times first in from the cutoff; if I always raise KK in that game state, the model will correctly assign the same absolute probability as other pocket pairs that I always raise.

From a human readability standpoint, I think this blue-red relative visualization is more informative than the orange absolute visualization from earlier. The blue-red relative visualization shows a player's strategy, and knowing an opponent's strategy is helpful for making human decisions while playing, whereas the orange absolute visualization shows probabilities that may be mathematically useful, but are less strategically useful in-game.

#### Bayes' Theorem Sneaks In

It may seem like magic that the HC-to-Act model can be "reverse engineered" to predict hole cards, but it's not magic, it's Bayes' theorem! Bayes' theorem is a way to derive a conditional probability (a probability that depends on something else happening) when you know a bunch of other probabilities.

In a given game state, when we ask what hole cards a player is likely holding after taking a particular action, we are asking about the probability of their hole cards given the observed action, or P(HC\|A). We don't know P(HC\|A), but we do know

- P(A\|HC): The probability of the player's action given particular hole cards. This is the HC-to-Act model output given a specific hole card hand.
- P(HC): The probability of a particular hole card hand, generally. This is just the naive probability of being dealt that hand.
- P(A): The probability of the player's action, generally (across all hole cards). This is just the sum of the HC-to-Act model output across all hole card hands.

Bayes theorem lets us put these probabilities together and solve for the one we don't know:

P(HC\|A) = P(A\|HC) \* P(HC) / P(A)

In a given game state, to determine the probability of a player holding a particular hole card card after taking a given action, P(HC\|A), we multiply the Act-to-HC model output, P(A\|HC), by the general probability of that hand, P(HC), and divide by the general probability of that action, P(A). This is what I was doing intuitively behind the scenes to translate the output of the Act-to-HC model into hole card probabilities. Neat!

## What's Next?

As part of this set of AI and machine learning projects, I am currently working on the following:

- Deploying these models to the cloud, where users can interact with them via a web browser
- Using Keras/Tensorflow to create a deep learning AI "poker bot" that can mimic my own and other players' complete strategies in a full game of poker. Consistent with ethical principle #2 (never while playing), the bot will not interface with any real-world poker websites, and will only operate in its own cloud instance for learning and entertainment purposes.

[![Poker bots](images/ml/robot_poker_by_theangryapple.jpg)](images/ml/robot_poker_by_theangryapple.jpg)

*"Robot Poker" by [theangryapple](https://www.deviantart.com/theangryapple/art/Robot-Poker-296743466) [\[Creative Commons license\]](http://creativecommons.org/licenses/by-sa/3.0/)*<br>

Consistent with ethical principle #1 (my own play first), my poker opponents should be very excited about the possibility of a poker bot that plays like me! Due to the asymmetry of known hole card data, AI-derived insights about my own play will always be more thorough than insights about other players. My opponents will be able to play against this very thorough Greg-bot as much as they'd like to practice their real-life strategies against me. You're welcome!

## The Take-Away Message

This project uses scikit-learn, Keras/TensorFlow, and other Python data science libraries to create a set of AI and machine learning models that reveal strategic insights about my own and my opponents' poker play. I have completed models that predict players' private hole cards, and am currently working on deploying these models to the cloud and developing a complete poker-playing deep learning model.
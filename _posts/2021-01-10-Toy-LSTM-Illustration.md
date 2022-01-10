---
title: "Toy LSTM Illustration"
layout: post
---

# What is an LSTM?

LSTM stands for Long Short-Term Memory network, a type of recurrent neural network used in machine learning. 

# Why should you care? 
LSTMs are used for automated discovery of relationships in time series data. Back in 2015, LSTMs were the latest and greatest thing for speech recognition and language translation. 

# How does it work?
I can't speak for how all LSTMs work, but I can speak for how this one LSTM does: 

| Component | Purpose |
|---        |---      |
| Memory       | A vector representing the hidden state of the system we are observing. |
| Eraser       | Tests the negative case for certain states, and resets them to zero in memory. |
| Writer       | Tests for state transitions and moves the memory to the new hidden state. |
| Write Filter | Tests for false positives in the writer and zeroes them. |
| Reader       | Tests for hidden state and predicts the next state transition. |
| Read Filter  | Tests for false positives in the reader and zeroes them. |

The defining feature of an LSTM is the 'memory'. Like a detective inspecting a crime scene, the LSTM processes each item in the sequence, tallying evidence for and against certain hypotheses, occasionally erasing previous theories as new information is revealed. Finally, the sum of the evidence points to a prediction of the next crime. 

# Reber Grammar
In order to gain an understanding of the inner workings of an LSTM, I hand-weighted a toy model to predict the next letter in a string of characters. The characters are generated using Reber Grammar. 

<p align="center">
    <img src= "../assets/images/reber-grammar.png" width="850" height="450" align="middle"/>
</p>

Reber Grammar is a rule set for generating strings of text. Starting with the letter B, we move through the graph, emitting letters as we cross each edge, yielding strings like BTXSE, BPVVE, or BTSSXXTTVPXVVE. 

# Toy LSTM

> Hopefully I have not made any horrible errors in my implementation. Please leave a comment if you want to share your knowledge and help me improve.

WIP.

# References
- https://colah.github.io/posts/2015-08-Understanding-LSTMs/
- http://www.bioinf.jku.at/publications/older/2604.pdf
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

Each time step, the LSTM receives a one-hot vector input representing the last emitted token. 

```js
let input   = [1,0,0,0,0,0,0]; //current token [B,T,S,X,P,V,E]
```

The indexes of the `memory`, `eraser`, `writer`, `filter`, and `reader` correspond to the index of the nodes in the Reber Grammar graph. 

```js
let memory  = [1,0,0,0,0,0]; //current node in the graph: [0,1,2,3,4,5]
```

For the sake of clarity, I show the `erase`, `write`, `filter` operations consecutively on each node. 

```js
//test for node 5: triggers on BP and X but not BX
//basically we tally up dependencies leading to this node
//using the filter to weed out false positives as required
//and reset the breadcrumbs if we cross an S or V, indicating we've left the node

eraser[5] = 1 / (1 + exp(-10 * (0.5 - S - V)));                 //reset on S,V
writer[5] = Math.tanh(0.55 * B + 0.7 * P + 5 * X);              //breadcrumbs to node 5
filter[5] = 1 / (1 + exp(-30 * (0.65 - memory[1])));            //do not increment from node 1

memory[5] = memory[5] * eraser[5] + writer[5] * filter[5]; 
```

> If you aren't familiar with the logistic and tanh formuals above, you can plug them into [desmos](https://www.desmos.com/calculator) to see what they look like. Basically the logistic function lets you do a boolean test for whether a vector is above/below a plane in n-space, and the tanh function performs the same test but in the range -1 to 1, so you can increment or decrement the memory based on the result. 

Note that the test for each node requires gaming the order of the edges preceding it. It is an extremely flaky way of coding, but that seems to align with my general impression that neural networks are doing blackbox code golf that is never meant to be read. It would be nice to go up a level on the ladder of abstraction, to a graph of all possible programs, connected by edges which are edits to the program; I think this would naturally converge on a basin of non-flaky code... but anyway. 

Here is the [full implementation](https://openprocessing.org/sketch/1412417):

```js

//this fails on BTSSXXTTTTTTTTTTTTTTTTTTTTTTTTTTTTSXVVE because filter[1] is not quiiiiiite zero

const tokens  = "BTSXPVE"; 
let input   = [1,0,0,0,0,0,0]; //current token [B,T,S,X,P,V,E]
let memory  = [0,0,0,0,0,0]; //one-hot TANH vector of current node in graph
let eraser  = [0,0,0,0,0,0]; //what to erase in memory
let writer  = [0,0,0,0,0,0]; //what to write to memory
let filter  = [1,1,1,1,1,1]; //filter the writer when it returns multiple write values
let reader  = [0,0,0,0,0,0,0]; //predicted tokens [B,T,S,X,P,V,E,-]

let str = 'B';

while(str.length < len)
{
    //memory accumulates evidence of a node being reached (sum of precedent edges)
    //writer adds evidence to memory when precedent edges are crossed
    //filter weeds out false positives in the writer
    //eraser resets memory when required
    //there are many many solutions in weight-space, this is just one of them
  			
    let B = input[0];
    let T = input[1];
    let S = input[2];
    let X = input[3];
    let P = input[4];
    let V = input[5];
    let E = input[6];

    //B
    eraser[0] = 0;                                                  //always erase
    writer[0] = Math.tanh(5 * B);                                   //increment on B, else do nothing 
    filter[0] = 1;

    //BT
    eraser[1] = 1 / (1 + exp(-10 * (0.5 - X - V - P)));             //reset on X,V,P
    writer[1] = Math.tanh(0.197 * B + 1.098 * T + 0.55 * S);        //breadcrumbs to node 1
    filter[1] = 1 / (1 + exp(-30 * (0.75 - memory[5])));            //do not increment from node 5

    //P_P, TX, X_P
    eraser[2] = 1 / (1 + exp(-30 * (0.65 - memory[2])));            //reset on exit
    writer[2] = Math.tanh(0.55 * (T + X + P));                      //breadcrumbs to node 2
    filter[2] = 1 / (1 + exp(-30 * (0.65 - memory[5])));            //do not increment from node 5

    //BP and X but not BX
    eraser[5] = 1 / (1 + exp(-10 * (0.5 - S - V)));                 //reset on S,V
    writer[5] = Math.tanh(0.55 * B + 0.7 * P + 5 * X);              //breadcrumbs to node 5
    filter[5] = 1 / (1 + exp(-30 * (0.65 - memory[1])));            //do not increment from node 1

    //PV, XV but not XX and not PX
    eraser[4] = 1 / (1 + exp(-10 * (0.6 - memory[4])));             //reset on exit
    writer[4] = Math.tanh(5 * V);                                   //breadcrumbs to node 4
    filter[4] = 1 / (1 + exp(-10 * (0.6 - memory[4])));             //filter V on exit

    //S (filter node 1) or VV
    eraser[3] = 1 / (1 + exp(-10 * (0.5 - P)));                     //reset on P
    writer[3] = Math.tanh(3.0 * S + 0.55 * V);                      //breadcrumbs to node 3
    filter[3] = 1 / (1 + exp(-10 * (0.9 - memory[1])));             //do not increment from node 1

    for(let i = 0; i < memory.length; i++) 
    { 
        memory[i] = memory[i] * eraser[i] + writer[i] * filter[i]; 
    }

    //|thresholds |false        |true |
    //|---        |---          |---  |
    //| memory[0] | 0.0         | 1.0 |
    //| memory[1] | 0.2         | 1.0 |
    //| memory[2] | 0.5         | 1.0 |
    //| memory[3] | 0.5         | 1.0 |
    //| memory[4] | 0.2         | 1.0 |
    //| memory[5] | 0.4,0.5,0.6 | 1.0 |
    //| memory[6] | 0.0         | 1.0 |

    reader[0]    = 0; //we never yield B
    reader[1]    = Math.tanh(5  * (memory[0] + memory[5] - 0.7)); //T may yield from 0 or 5
    reader[2]    = Math.tanh(5  * (memory[1] + memory[2] - 0.7)); //S may yield from 1 or 2
    reader[3]    = Math.tanh(5  * (memory[1] + memory[2] - 0.7)); //X may yield from 1 or 2
    reader[4]    = Math.tanh(5  * (memory[0] + memory[4] - 0.7)); //P may yield from 0 or 4 
    reader[5]    = Math.tanh(10 * (memory[4] + memory[5] - 0.9)); //V may yield from 4 or 5
    reader[6]    = Math.tanh(5  * (memory[3]             - 0.7)); //E may yield from 3

    //no read filter layer, not required.

    let c = reader
        .map((e, i) => [e, i]) 		
        .filter(e => e[0] > 0.5) 	
        .map(e => e[1]); 					
    ;
    if(c.length == 0) console.log("error error error");
    let newToken = random(c);

    //append token to string
    str += tokens[newToken]; 

    if(debug) 
    {
        console.log("memory 2: " + memory.map(e => round(e,2)).join()); //round(1E-8) returns NaN in p5
        console.log("reader:   " + reader.map(e => round(e,2)).join());
        console.log(str);
        console.log("------");
    }

    //graph exit condition
    if(newToken == 6) break; 

    //set next input
    input = [0,0,0,0,0,0,0];
    input[newToken] = 1;
  
}

return str;

```
# Conclusions

This code is definitely not perfect. It fails on `BTSSXXTTTTTTTTTTTTTTTTTTTTTTTTTTTTSXVVE` because `filter[1]` doesn't return exactly zero when blocking false positives. 

I also make a starting assumption that all strings start with B, whereas real-world historical data will start in some unknown hidden state. The LSTM may need to accumulate breadcrumbs on multiple nodes until it gets a clear signal about which part of the graph it is in. This sounds equivalent to adding new edges to the graph, like 'move to node 2 if you get input X from node 5'.

Technically I should train an LSTM on the embedded Reber Grammar next (apparently its the Hello World of LSTMs), we'll see if I get time. 

# References
- [https://colah.github.io/posts/2015-08-Understanding-LSTMs/](https://colah.github.io/posts/2015-08-Understanding-LSTMs/)
- [http://www.bioinf.jku.at/publications/older/2604.pdf](http://www.bioinf.jku.at/publications/older/2604.pdf)
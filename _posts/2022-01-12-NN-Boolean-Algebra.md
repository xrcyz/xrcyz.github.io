---
title: "Neural Network Boolean Algera"
layout: post
---

In this post I explore the idea of using boolean algebra in neural networks to construct arbitrary solids in the input space. I found this mental model useful for reasoning about how neural networks perform their functions. 

## Demo

I sketched up a quick [proof of concept](https://xrcyz.github.io/neural-metaballs/) to go with this post. It uses a three layer network to create a metaball heightmap, and animates the metaballs by changing the biases in layer 1. 

<p align="center">
    <a href="https://xrcyz.github.io/neural-metaballs/">
        <img src= "../assets/images/neural-metaballs.gif" alt="neural metaballs" width="900" height="450" align="middle"/>
    </a>
</p>

## Artificial Neurons

The name 'artificial neuron' doesn't tell us a lot about what it is or what it does. Instead, let's look at some code.

```js
let activation = 1 / (1 + exp(crossproduct(weights, inputs) + bias));
```

There are two important things happening in the activation function. 
1. The sum of the cross product with the bias is equivalent to testing if a point is above a hyperplane. 
2. The logistic function converts the test to a boolean (kinda). 

### Hyperplanes

The value of `crossproduct(weights, inputs) + bias` is equivalent to testing if a point is above a line/plane. 

For one-dimensional inputs, `crossproduct(weights, inputs) + bias` is equal to `m * x + c`. If this value is positive, then the input must be above the threshold `x = -c / m`. Likewise if the number is negative, then the input must be below the threshold `x = -c / m`.

```js
let input = x;
let weight = m;
let bias = c;
let test = 1 / (1 + exp(m * x + c));
```

For two-dimensional inputs, `crossproduct(weights, inputs) + bias` is equal to `a * x + b * y + c`. If this number is positive, then the input must be above the line `y = (-a * x - c) / b`. Likewise if the number is negative, then the input must be below the line `y = (-a * x - c) / b`.

```js
let inputs = [x,y];
let weight = [a,b];
let bias = c;
let test = 1 / (1 + exp(a * x + b * y + c));
```

Since we can extend this to planes in arbitrary dimensions, it is safe to say that `crossproduct(weights, inputs) + bias`  always tests for if the input vector is above or below a specified plane. 

### Boundary slope

When an input vector travels from below the hyperplane to above the hyperplane, the value of the logistic function transitions from zero to one. The slope of this transition is defined by the scale of the weights and the bias. 

```js
let input  = x;
let weight = scale * m;
let bias   = scale * c;
let test = 1 / (1 + exp(scale * (m * x + c)));
```

Note that the scale does not alter the definition of the hyperplane. We still calculate the term `m * x + c`, returning a positive or negative number. Then the `scale` parameter is used to sharpen or soften the logistic slope as we pass through that plane. 

## Layer 0: Inputs

The neural network input layer holds an array of inputs. These are equivalent to the parameters of a method call. Ideally the inputs are normalised between zero and one, so the input vectors are in a predictable range. 

## Layer 1: Linearly Separating the Input Space

Assuming normalised inputs, then the input vector is guaranteed to reside within the unit hypercube of the input space. 
- 1 input => a vector on a unit line.
- 2 inputs => a vector inside a unit square.
- 3 inputs => a vector inside a unit cube.
- N inputs => a vector inside a unit hypercube.

A single neuron in layer 1 can test for conditions by linearly separating the line, square, or cube. 

In one dimension: 
```js
let test = 1 / (1 + exp(10 * (x - 0.5))); //return (x > 0.5);
```

In three dimensions: 
```js
let inputs = [x,y,z];
let test = 1 / (1 + exp(-10*(x + y - z - 1.5))); //return  (x && y && !z); equivalent to the plane z = x + y - 1.5 isolating vertex (1,1,0)
```

I want to stress that this is a _geometric_ operation. In the case above, the neuron defines an implicit plane `z = x + y - 1.5` that linearly separates the unit cube. We can think of the input cube as being sliced into two solids, one red and one blue. If a point falls in the red or the blue solid, it receives a different classification.

(We could also consider this to be a kind of heightmap, where the 'solids' are contours or isosurfaces on that heightmap. When we round the predictions of a neural network to one or zero, that is equivalent to running a contour line at height 0.5 through the heightmap defined by the neural network.)

## Layer 2: Convex Hulls in Input Space

Layer 2 lets us perform union operations on the solids defined in layer 1. This lets us create any convex hull in the input space.

(Technically we can do intersect and subtract operations in this layer as well, but since we are only dealing with linear separations, it works out to the same thing.)

```js
//input layer
let inputs = [x,y];

//layer 1
let L1 = 
{
    a: 1 / (1 + exp(-10 * (x - 0.5))), //return (x > 0.5)
    b: 1 / (1 + exp(-10 * (y - 0.5))), //return (y > 0.5)
};

//layer 2
let L2 = 
{
    a: 1 / (1 + exp(-10 * (L1.a + L1.b - 1.5))), //return (x > 0.5 && y > 0.5)
}
```

To expand on this, we need to distinguish between the __input space__ and the __layer 1 space__. 
- __input space__ is a unit square containing `[x,y]`. 
- __L1 space__ is a a unit square containing `[L1.a,L1.b]`.

We know that each neuron in `L1` performs a single linear separation in the __input space__. 
- `L1.a` slices the unit cube halfway along the x-axis, colouring the solids red/blue for above/below the plane;
- `L1.b` slices the unit cube halfway long the y-axis, colouring the solids red/blue for above/below the plane;

| Layer 1 Output  | Solid Intersections |
|---              |---                  |
| `[0,0]`         | BLUE, BLUE          |
| `[1,0]`         | RED, BLUE           |
| `[0,1]`         | BLUE, RED           |
| `[1,1]`         | RED, RED            |

The neurons in layer 2 are now able to linearly separate the __layer 1 space__, which allows us to test for combinations of true/false values. For example, with the line `y = 1.5 - x` we can test if a point in __input space__ lies in the intersecting solids `RED, RED`. 

Using this method, a single neuron in layer 2 should be able to construct any convex hull in the input space. 
- In layer 1, each neuron defines a line/plane with a normal direction, `RED` pointing to inside the hull.
- In layer 2, any point that tests `[RED,RED,...,RED]` is inside the hull.

Concave hulls cannot be built using this method, since a point inside a cave would return `RED` for lines/planes that are part of the convex hull. 

## Layer 3: Concave Hulls in Input Space

In Layer 3 we may union or subtract solids from layer 2. This lets us test any concave hulls with voids in the input space.

```js
//input layer
let inputs = [x,y]; //point in the unit square

//layer 1 - linear separation by line/plane
let L1 = 
{
    a: 1 / (1 + exp(-10 * (x - 0.1))), //return (x > 0.1)
    b: 1 / (1 + exp(-10 * (x - 0.3))), //return (x > 0.3)
    c: 1 / (1 + exp(-10 * (0.7 - x))), //return (x < 0.7)
    d: 1 / (1 + exp(-10 * (0.9 - x))), //return (x < 0.9)
    e: 1 / (1 + exp(-10 * (y - 0.1))), //return (y > 0.1)
    f: 1 / (1 + exp(-10 * (y - 0.3))), //return (y > 0.3)
    g: 1 / (1 + exp(-10 * (0.7 - y))), //return (y < 0.7)
    h: 1 / (1 + exp(-10 * (0.9 - y))), //return (y < 0.9)
};

//layer 2 - convex hulls by intersecting normals
let L2 = 
{
    a: 1 / (1 + exp(-10 * (L1.a + L1.d + L1.e + L1.h - 3.5))), //big square
    b: 1 / (1 + exp(-10 * (L1.b + L1.c + L1.f + L1.g - 3.5))), //little square
}

//layer 3 - concave hulls by subtracting convex hulls
let L3 = 
{
    a: 1 / (1 + exp(-10 * (L2.a - L2.b - 0.5))), //square donut
}
```

The same principles apply here. The neuron `L3.a` is linearly separating __layer 2 space__ to test if the input point is inside the big square and outside the little square. 

## Layer 4: Arbitrary Regions in Input Space

In layer 4 we can union isolated solids from layer 3. This lets us test any arbitrary region of the input space. 

```js
//input layer
let inputs = [x,y]; //point in the unit square

//layer 1 - linear separation by line/plane
let L1 = 
{
    a: 1 / (1 + exp(-10 * (x - 0.1))), //return (x > 0.1)
    b: 1 / (1 + exp(-10 * (x - 0.2))), //return (x > 0.2)
    c: 1 / (1 + exp(-10 * (0.3 - x))), //return (x < 0.3)
    d: 1 / (1 + exp(-10 * (0.4 - x))), //return (x < 0.4)

    e: 1 / (1 + exp(-10 * (y - 0.1))), //return (y > 0.1)
    f: 1 / (1 + exp(-10 * (y - 0.2))), //return (y > 0.2)
    g: 1 / (1 + exp(-10 * (0.3 - y))), //return (y < 0.3)
    h: 1 / (1 + exp(-10 * (0.4 - y))), //return (y < 0.4)

    i: 1 / (1 + exp(-10 * (x - 0.6))), //return (x > 0.6)
    j: 1 / (1 + exp(-10 * (x - 0.7))), //return (x > 0.7)
    k: 1 / (1 + exp(-10 * (0.8 - x))), //return (x < 0.8)
    m: 1 / (1 + exp(-10 * (0.9 - x))), //return (x < 0.9)

    n: 1 / (1 + exp(-10 * (y - 0.6))), //return (y > 0.6)
    p: 1 / (1 + exp(-10 * (y - 0.7))), //return (y > 0.7)
    q: 1 / (1 + exp(-10 * (0.8 - y))), //return (y < 0.8)
    r: 1 / (1 + exp(-10 * (0.9 - y))), //return (y < 0.9)
};

//layer 2 - convex hulls by intersecting normals
let L2 = 
{
    a: 1 / (1 + exp(-10 * (L1.a + L1.d + L1.e + L1.h - 3.5))), //big square 1
    b: 1 / (1 + exp(-10 * (L1.b + L1.c + L1.f + L1.g - 3.5))), //little square 1
    c: 1 / (1 + exp(-10 * (L1.i + L1.m + L1.n + L1.r - 3.5))), //big square 2
    d: 1 / (1 + exp(-10 * (L1.j + L1.k + L1.p + L1.q - 3.5))), //little square 2
}

//layer 3 - concave hulls by subtracting convex hulls
let L3 = 
{
    a: 1 / (1 + exp(-10 * (L2.a - L2.b - 0.5))), //square donut 1 
    b: 1 / (1 + exp(-10 * (L2.c - L2.d - 0.5))), //square donut 2
}

//layer 4 - any region by unioning concave hulls
let L4 = 
{
    a: 1 / (1 + exp(-10 * (L3.a + L3.b - 0.5))), //square donut 1 and square donut 2
}
```

In this example, the neuron `L4.a` is linearly separating __layer 3 space__ to test if the input point is inside either one of the two square donuts. 

## The Language of Neural Networks 

Armed with this mental model, we can begin to reason about how neural networks perform their functions. 

Suppose you have a program you want to model as a neural network. If we can find a schema to represent the input parameters as numbers between one and zero, then we can represent the set of all program test cases as a point cloud in a unit hypercube. Similarly, if we can represent all components of the return value of the function as numbers between one and zero, then each one of those could be a neuron in the output layer of a network. Now we can use boolean algebra to construct arbitrary hypersolids in the input space, drawing hulls around sets of inputs that return the same answer, thus encoding the logic of our program in the weights of the network. 

If we shrink-wrap the hulls too tightly, then we get "over training". Ideally we want the program to be able to interpolate similar answers to similar inputs, so that it doesn't fail on edge cases that weren't in the training data set (the test regime). 


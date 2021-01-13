# The Convolutional Tsetlin Machine
The Convolutional Tsetlin Machine (<a href="https://arxiv.org/abs/1905.09688">https://arxiv.org/abs/1905.09688</a>) learns interpretable filters using propositional formulae. The propositional formulae are composed by a collective of Tsetlin Automata coordinated through a game. The Convolutional Tsetlin Machine is an interpretable alternative to convolutional neural networks.

## Implementation

* Multi-threaded implementation of the Tsetlin Machine, Convolutional Tsetlin Machine, Regression Tsetlin Machine, and Weighted Tsetlin Machine, with support for continuous features and multi-granular clauses, https://github.com/cair/pyTsetlinMachineParallel, https://pypi.org/project/pyTsetlinMachineParallel/

* High-level Tsetlin Machine Python API with fast C-extensions. Implements the Tsetlin Machine, Convolutional Tsetlin Machine, Regression Tsetlin Machine, and Weighted Tsetlin Machine, with support for continuous features, multi-granular clauses, and clause indexing, https://github.com/cair/pyTsetlinMachine, https://pypi.org/project/pyTsetlinMachine/

## Tutorial

### Example Problem: 2D Noisy XOR

<p>
In this tutorial, I will use the 2D Noisy XOR dataset to demonstrate how the Convolutional Tsetlin Machine recognizes patterns and how these patterns are learnt from example images. The dataset I consider contains 3x3 binary images. Below you see an example image.
</p>
<p align="center">
  <img width="20%" src="https://github.com/olegranmo/blob/blob/master/Example_Image.png">
</p>

<p>
The 9 bits of the image are assigned coordinates (x, y), so that the upper left bit is at position (1, 1), the bit to its right is at position (2, 1), while the bit below it is at position (1, 2), and so on. The 9 bit values are randomly set for each image, except for the four bits of the 2x2 patch in the upper right corner (marked by green bit values). These four bit values reveal the class of the image:
</p>
<p align="center">
  <img width="25%" src="https://github.com/olegranmo/blob/blob/master/Patterns.png">
</p>
<p>
A horizontal line is associated with class 0, while a diagonal line is associated with class 1. The dataset thus captures a 2D version of the XOR-relation.
</p>
<p>
For this example, I consider convolution with 2x2 filters. A convolutional learning mechanism employing 2x2 filters must learn the four patterns above as well as which class they belong to. Note that due to the XOR-relation, linear classifiers will face difficulties handling this task.
</p>
<p>
A 3x3 image contains four distinct 2x2 patches, located at different coordinates (x, y) within the image. One is located in the upper left corner of the image, at position (1, 1), another at position (2, 1), a third at position (1, 2), and the fourth at position (2, 2). The content of each patch is modelled with four propositional variables <img src="http://latex.codecogs.com/svg.latex?\mathbf{X} = [x_{1,1}, x_{2,1}, x_{1,2}, x_{2,2}]" border="0"/>. The coordinates of the variables (lower index) are the relative positions of the variables within the patch: 
<p align="center">
  <img width="10%" src="https://github.com/olegranmo/blob/blob/master/Filter.png">
</p>
<p>
Which image bit a propositional variable refers to thus depends both on the coordinates of the variable within the patch and the position of the patch within the image. As an example, the variable <img src="http://latex.codecogs.com/svg.latex?x_{1,1}" border="0"/> of the upper right patch refers to the bit at position (2, 1) in the 3x3 image.
</p>

<p>
I will now explain how the Convolutional Tsetlin Machine solves the above pattern recognition task, going through the recognition and learning steps in detail.
</p>

<p align="center">
  <img width="20%" src="https://github.com/olegranmo/blob/blob/master/Michael_Lvovitch_Tsetlin.png">
</p>
<p align="center">
  <b>Michael Lvovitch Tsetlin (22 September 1924 – 30 May 1966)</b>
</p>

### The Tsetlin Automaton

<p>
The Convolutional Tsetlin Machine is based on the Tsetlin Automaton, introduced by M. L. Tsetlin in 1961. The Tsetlin Automaton is one of the pioneering solutions to the well-known multi-armed bandit problem and the first Finite State Learning Automaton.
</p>

#### Two-Action Tsetlin Automata
<p>
A two-action Tsetlin Automaton chooses among two actions, Action 1 or Action 2, and performs these sequentially in an environment. For each action performed, the environment responds stochastically with a Penalty or a Reward, according to an unknown reward probability distribution <img src="http://latex.codecogs.com/svg.latex?R=\[r_1, r_2\]" border="0"/>. When Action 1 is performed, the environment responds with a Reward with probability <img src="http://latex.codecogs.com/svg.latex?r_1" border="0"/>, otherwise, it responds with a Penalty. For Action 2, the probability of Reward is <img src="http://latex.codecogs.com/svg.latex?r_2" border="0"/>.  By interacting with the environment, the goal of the Tsetlin Automaton is to, as quickly as possible, single in on the action that has the highest Reward probability.
</p>

<p>
A Tsetlin Automaton is a finite state machine. Below you see a two-action Tsetlin Automaton with 6 states, 3 states per action.
</p>
<p align="center">
  <img width="65%" src="https://github.com/olegranmo/blob/blob/master/fixed_deterministic_run_1.png">
</p>
<p>
When the automaton is in states 1-3 (left half) it performs Action 1, and when it is in states 4-6 (right half) it performs Action 2.
</p>

<p>
The Tsetlin Automaton learns by changing state. Each state transition is decided by the feedback from the environment (Penalty or Reward). As shown in the figure above, a Penalty makes the Tsetlin Automaton change state towards the centre, while a Reward makes it change state away from the centre. In effect, the farther the Tsetlin Automaton is away from the centre, the more confident it is in the action chosen.
</p>

#### Example Run
<p>
The Tsetlin Automaton depicted above is in state 3 (marked with a solid black circle). Accordingly, it selects Action 1. Assume this triggers a Penalty from the environment. The Tsetlin Automaton then moves from state 3 to state 4:
</p>

<p align="center">
  <img width="65%" src="https://github.com/olegranmo/blob/blob/master/fixed_deterministic_run_2.png">
</p>
It is now in the right half of states and therefore selects Action 2. This time, the Tsetlin Automaton receives a Reward, updating its state accordingly:
<p align="center">
  <img width="65%" src="https://github.com/olegranmo/blob/blob/master/fixed_deterministic_run_3.png">
</p>
<p>
At this point, it is quite confident that Action 2 is better than Action 1. Indeed, at least two consecutive Penalties are needed to make the Tsetlin Automaton change its mind and switch back to performing Action 1 again.
</p>

<p>
The above simple learning mechanism has some remarkable properties. It makes the Tsetlin Automaton act predictably, only changing action when switching between the two centre states. This supports stable collectives of many cooperating Tsetlin Automata, taking part in solving more complex problems. Further, the Tsetlin Automaton never stops learning. Therefore, it can adapt to changes in the environment, for instance caused by other Tsetlin Automata.  Finally, the accuracy and speed of learning are controlled by the number of states. By increasing the number of states and learning cycles towards infinity, the Tsetlin Automaton performs the optimal action with probability arbitrarily close to unity. In other words, Tsetlin Automata learning is asymptotically optimal.
</p>

### The Architecture of the Convolutional Tsetlin Machine

#### Overview of Architecture
<p>
The structure of the classic Tsetlin Machine is shown below. This structure forms the core of the Convolutional Tsetlin Machine:
</p>
<p align="center">
  <img width="40%" src="https://github.com/olegranmo/blob/blob/master/Overall_Architecture.png">
</p>
<p>
In our 2D Noisy XOR example, the Convolutional Tsetlin Machine takes the propositional variables <img src="http://latex.codecogs.com/svg.latex?x_{1,1}, x_{2,1}, x_{1,2}, x_{2,2}" border="0"/> and their negations <img src="http://latex.codecogs.com/svg.latex?\lnot{x_{1,1}}, \lnot{x_{2,1}}, \lnot{x_{1,2}}, \lnot{x_{2,2}}" border="0"/> as input. These inputs are referred to as literals, forming a bit vector. The input is fed to a fixed number of conjunctive clauses, how many is decided by the user. These clauses evaluate to either 0 or 1. Further, half of the clauses are assigned negative polarity (-), while the other half is assigned positive polarity (+). The polarity decides whether the clause output is negative or positive. Negative clauses are used to recognize class y=0, while positive clauses are used to recognize class y=1. Next, a summation operator aggregates the output of the clauses. The final classification is performed with a threshold function. Class y=1 is predicted if the number of positive clauses outputting 1 exceeds or equals the number of negative clauses outputting 1. Otherwise, class y=0 is predicted.
</p>

#### The Conjunctive Clause
<p>
Each conjunctive clause above is built by ANDing a selection of the available propositional variables <img src="http://latex.codecogs.com/svg.latex?x_{1,1}, x_{2,1}, x_{1,2}, x_{2,2}" border="0"/> and their negations <img src="http://latex.codecogs.com/svg.latex?\lnot{x_{1,1}}, \lnot{x_{2,1}}, \lnot{x_{1,2}}, \lnot{x_{2,2}}" border="0"/>. The clause <img src="http://latex.codecogs.com/svg.latex?C = {x_{1,1}} \land {x_{2,2}} \land  \lnot{x_{2,1}} \land \lnot{x_{1,2}}" border="0"/>, for instance, evaluates to 1 for image patches with bit values:
</p>
<p align="center">
  <img width="10%" src="https://github.com/olegranmo/blob/blob/master/y_1a.png">
</p>
and to 0 for other image patches.

#### The Tsetlin Automata Team for Composing Clauses
<p>
Each clause of the Convolutional Tsetlin Machine is composed by a team of Tsetlin Automata. These decide which literals are to be Excluded from the clause and which are to be Included. There is one Tsetlin Automaton per literal, deciding upon its inclusion:
<p>
<p align="center">
  <img width="90%" src="https://github.com/olegranmo/blob/blob/master/Example_Configuration_1a.png">
</p>
<p>
The team in the figure has for instance decided to include <img src="http://latex.codecogs.com/svg.latex?{x_{1,1}}, {x_{2,2}}, \lnot{x_{2,1}}" border="0"/> and <img src="http://latex.codecogs.com/svg.latex?\lnot{x_{1,2}}" border="0"/>, producing the clause <img src="http://latex.codecogs.com/svg.latex?C = {x_{1,1}} \land {x_{2,2}} \land \lnot{x_{2,1}} \land \lnot{x_{1,2}}" border="0"/>.
</p>

### Recognition with the Convolutional Tsetlin Machine

#### Clause Convolution Step
<p>
The Convolutional Tsetlin Machine recognizes patterns by first turning the input image into patches. For our 2D Noisy XOR example, the 3x3 input image is turned into four 2x2 patches:
</p>
<p align="center">
  <img width="60%" src="https://github.com/olegranmo/blob/blob/master/Convolution_Example.png">
</p>
<p>
Each conjunctive clause is then evaluated on each patch. For each clause, the outcome for each patch is ORed to produce the output of the clause. The figure shows this procedure for one of the clauses in our 2D Noisy XOR example.
</p>
<p>
To make the clauses location-aware, each patch is further enhanced with its coordinates within the image (see figure). Location awareness may prove useful in applications where both patterns and their location are distinguishing features, e.g. recognition of facial features such as eyes, eyebrows, nose, mouth, etc. in facial expression recognition. These coordinates are incorporated as additional propositional variables in the input vector. However, for the sake of brevity, I will not consider the details of this incorporation here. Instead, I will simply assume that the information on the coordinates already have been incorporated into the clauses.
</p>

#### Summation and Thresholding Step
<p>
In our example architecture, there are eight conjunctive clauses:
</p>
<p align="center">
  <img width="65%" src="https://github.com/olegranmo/blob/blob/master/Recognition.png">
</p>
<p>
The above configuration consists of four positive clauses which represent XOR patterns. These are used to recognize images of class y=1. It also consists of four negative clauses which represent patterns associated with class y=0.  Observe that each clause has been annoted with the positional information it has incorporated, using thresholds on the x and y coordinates. As explained earlier, the bit patterns inside each clause have been decided by the eight corresponding Tsetlin Automata, one per literal in the 2x2 filter.
</p>
<p>
Our example clause is highlighted in the figure, outputting 1 due to matching the input image. Note that some of the other clauses are matching the image content too. However, their positional information is incompatible with the input, so they output 0. As in the classic Tsetlin Machine, the output from each clause is processed further by summation and then thresholding to decide the class. Above, positive clauses outputting 1 are in majority, so the Convolutional Tsetlin Machine assigns class y=1 to the input image.
</p>

### Learning with the Convolutional Tsetlin Machine

We are now ready to address how the Convolutional Tsetlin Machine learns. For the sake of clarity, I will first consider learning of positive clauses, returning to the negative clauses towards the end of the section.

#### Allocation of Pattern Representation Resources

<p>
Each clause can be seen as a resource for representing patterns. With limited resources, it is critical to allocate resources wisely. The Convolutional Tsetlin Machine seeks to allocate clauses uniformly among the crucial patterns in the dataset. This is achieved with a target value T. That is, each time the outputs of the clauses are summed up, T is the target value for the summation. For inputs of class y=0 the target value is -T and for inputs of class y=1 the target value is T. 
</p>
<p>The resources are allocated by controlling the intensity of the bandit learning feedback cycle. In brief, the feedback cycle is increasingly intensified the farther away the clause output is from the target value T. Conversely, feedback comes to a complete standstill when T is reached or exceeded. Let v denote the summed up clause output. Feedback intensity is modelled as the probability of activating each clause. For input of class y=0, the probability of activating a clause is:
<p align="center">
<img src="http://latex.codecogs.com/svg.latex?\frac{T + \mathrm{max}(-T, \mathrm{min}(T, v))}{2T}" border="0"/>
</p>
<p>
Using T=2 as an example, feedback intensity is 0.0 up to v=-2. Thereafter, it gradually increases, reaching 1.0 at v=2:
</p>
<p align="center">
  <img width="60%" src="https://github.com/olegranmo/blob/blob/master/Clause_Activation_Probability_y0.png">
</p>
<p>
For input of class y=1, the clause activation probability is:
</p>
<p align="center">
<img src="http://latex.codecogs.com/svg.latex?\frac{T - \mathrm{max}(-T, \mathrm{min}(T, v))}{2T}" border="0"/>
</p>
<p>
Then, with T=2, feedback intensity is 1.0 up to v=-2. From v=-2, intensity drops, reaching 0.0 at v=2:
</p>
<p align="center">
  <img width="60%" src="https://github.com/olegranmo/blob/blob/master/Clause_Activation_Probability_y1.png">
</p>
<p>
If a clause is not activated, no feedback is given to the Tsetlin Automata of that clause.
</p>

<p>
<b>Remark 1.</b> Observe that the future returns of a clause are diminishing with the number of other clauses that capture the same sub-pattern. This is crucial for stimulating the Tsetlin Automata teams to distribute themselves across the critical sub-patterns. 
</p>

<p>
<b>Remark 2.</b> A larger T (with a corresponding increase in the number of clauses) makes the learning more robust. This is because more clauses are involved in learning each specific sub-pattern, introducing an ensemble effect.
</p>
<p>
Now, consider the Convolutional Tsetlin Machine configuration below. The Noisy 2D XOR problem has almost been solved. However, there is an imbalance in the representation of class y=1. Three clauses are allocated to represent one of the sub-patterns, while only a single clause has been allocated to represent the remaining sub-pattern. 
</p>
<p align="center">
  <img width="105%" src="https://github.com/olegranmo/blob/blob/master/Learning.png">
</p>
<p>
For this example I use the target value T=2. When an input image of the overrepresented sub-pattern appears, nothing happens because T is exceeded by the three votes from the matching positive clauses. However, when an input image of the underrepresented sub-pattern comes along, as shown in the figure, only a single clause outputs 1. It is the clause highlighted in the figure that matches the image (upper right patch) and accordingly outputs 1. In this case, none of the negative clauses respond since their patterns do not match the image either (being constrained by the positional information).
</p>
<p>
Thus, the Convolutional Tsetlin Machine’s combined output is v=1. Learning of feature detectors proceeds as follows. With the target value set to T=2, the probability of feedback is (T-v)/(2T)=0.25, and thus learning taking place. This pushes the output v towards T=2. We achieve this with what we refer to as Type I feedback.
</p>

#### Type I Feedback 
<p>
Type I feedback reinforces true positive output and reduces false negative output. That is, it makes the Convolutional Tsetlin Machine output 1 when it should output 1. Type I feedback subdivides into Type Ia and Type Ib feedback. Type Ia feedback is given when a clause outputs 1, and is the part that reinforces true positive output. Type 1b feedback, on the other hand, is given when a clause outputs 0, combating false negative output.
</p>

##### Type Ib Feedback

<p>
In accordance with the flow of the example, let us consider Type Ib feedback first. We further consider what happens when the underrepresented sub-pattern appears. The three positive clauses that capture the overrepresented sub-pattern then all output 0. Here is one of them: 
</p> 

<p align="center">
  <img width="90%" src="https://github.com/olegranmo/blob/blob/master/Example_Configuration_4a.png">
</p>
<p>
To balance usage of pattern representation resources, one of the three clauses should instead output 1. The goal of Type Ib feedback is to make that happen by forcing one of the three clauses to reorganize. This is achieved by penalizing all Include actions and rewarding all Exclude actions of the afflicted clauses. However, for many problems, applying Type Ib feedback with full force will be too disruptive, erasing the patterns captured by the clauses too quickly. Therefore, the impact of Type Ib feedback is reduced by a factor s. That is, each Tsetlin Automaton is not deterministically penalized/rewarded, but stochastically with probability:
</p>
<p align="center">
<img src="http://latex.codecogs.com/svg.latex?\frac{1}{s}" border="0"/>
</p>
Additionally, this mechanism combats overfitting, because a smaller s forces focusing on more frequent patterns. In effect, the underlying noise is "forgotten" by the persistent reinforcement of Exclude actions. A larger s, on the other hand, provides finer patterns.

##### Type Ia Feedback

<p>
Let us now consider Type Ia feedback. Eventually, Type Ib feedback makes an additional clause to recognize the underrepresented pattern in our example:
</p> 
<p align="center">
  <img width="10%" src="https://github.com/olegranmo/blob/blob/master/y_1a.png">
</p>
<p>
This happens because Type Ib feedback is applied persistently every time the underrepresented sub-pattern appears in an image, while the overrepresented sub-pattern causes no change at all.  Exclude actions are reinforced, while Include actions are suppressed. Thus, sooner or later, one of the three clauses will start to recognize the underrepresented pattern. Assume the following clause is the first one:
</p> 

<p align="center">
  <img width="90%" src="https://github.com/olegranmo/blob/blob/master/Example_Configuration_2a.png">
</p>

<p>
As seen, due to the repeated application of Type Ib feedback, all actions are now Exclude and the clause outputs 1 for any pattern. For this particular clause, all of the literals that were included initially were in conflict with the underrepresented pattern. Therefore, they all had to be excluded to prepare the clause for learning. We are now ready to introduce Feedback Type Ia.
</p>

<p>
Type Ia feedback reinforces patterns when they are recognized. That is, Type Ia feedback is given to clauses that output 1. As a first step, one of the image patches that made the clause evaluate to 1 is randomly selected. For our example, let us assume the following patch is selected:
<p align="center">
  <img width="10%" src="https://github.com/olegranmo/blob/blob/master/y_1a.png">
</p>
<p>
It is the values of the corresponding propositional variables <img src="http://latex.codecogs.com/svg.latex?x_{1,1}, x_{2,1}, x_{1,2}, x_{2,2}" border="0"/> and their negations <img src="http://latex.codecogs.com/svg.latex?\lnot{x_{1,1}}, \lnot{x_{2,1}}, \lnot{x_{1,2}}, \lnot{x_{2,2}}" border="0"/> (the literals) that control Type Ia feedback. In all brevity, literals of value 1 in the selected image patch reinforce Include actions (penalize Exclude and reward Include). Literals of value 0, on the other hand, reinforce Exclude actions (reward Exclude and penalize Include).
</p>
<p>
Thus, for our example, the Include action is reinforced for  <img src="http://latex.codecogs.com/svg.latex?x_{1,1}, \lnot{x_{2,1}}, \lnot{x_{1,2}}, x_{2,2}" border="0"/> and the Exclude action is reinforced for <img src="http://latex.codecogs.com/svg.latex?\lnot{x_{1,1}}, x_{2,1}, x_{1,2}, \lnot{x_{2,2}}" border="0"/>. The latter reinforcement happens every time the underrepresented pattern appears in an image. Eventually, the literals <img src="http://latex.codecogs.com/svg.latex?x_{1,1}, \lnot{x_{2,1}}, \lnot{x_{1,2}}, x_{2,2}" border="0"/> are included in the clause. Assume that <img src="http://latex.codecogs.com/svg.latex?x_{1,1}" border="0"/> is included first:
</p>
<p align="center">
  <img width="90%" src="https://github.com/olegranmo/blob/blob/master/Example_Configuration_3a.png">
</p>

#### Type II Feedback 

<p>
Type II feedback further stimulates the capability of clauses to distinguish between class y=0 and class y=1. To achieve this, feedback of Type II is activated when clauses output 1 for input images of the other class. For example, the clause that we just considered will sometimes also recognize sub-patterns associated class y=0. By only including a single literal, <img src="http://latex.codecogs.com/svg.latex?x_{1,1}" border="0"/>, the pattern is too loosely specified. When this happens, Type II feedback is activated to combat the false positive output. Again, we randomly select one of the image patches that made the clause evaluate to 1. To force the clause to eventually output 0 for this particular sub-pattern, all the Tsetlin Automata with zero-valued literals are penalized for their Exclude decision. As a result, sooner or later, the offending sub-pattern will no longer be recognized because one of its zero-valued literals has been included in the clause!
</p>

By the combined effect of Type Ia and Type II feedback, the clause becomes sufficiently strict over time:
<p align="center">
  <img width="90%" src="https://github.com/olegranmo/blob/blob/master/Example_Configuration_1a.png">
</p>

<b>Remark.</b> All of the above operations are for positive clauses.  For negative clauses, Type I feedback is simply replaced with Type II feedback and vice versa!

### Goal State and Nash Equilibrium
<p>
The Convolutional Tsetlin Machine is now in its goal state, stable in a Nash Equilibrium. That is, no single Tsetlin Automaton will benefit from changing its action:
</p>

<p align="center">
  <img width="65%" src="https://github.com/olegranmo/blob/blob/master/Goal_State.png">
</p>

<p>
As seen, two clauses have been allocated to each sub-pattern. Thus, for any sub-pattern, the combined output of the complete set of clauses is 2 for y=1 and -2 for y=0. Thus, no further learning is necessary for the detection of the XOR sub-patterns.
</p>

The above example Nash Equilibrium is rather simple. It manifests when all the patterns are perfectly classified and the summed clause output reaches 2/-2. For more complex scenarios, the Nash Equilibria of the Convolutional Tsetlin Machine also balance false negative against false positive classifications, while combating overfitting. This happens when Type Ia, Type Ib and Type II feedback are in balance.

## Demo

```bash
pip install pyTsetlinMachine

./2DNoisyXORDemo.py 

#1 Mean Accuracy (%): 99.97; Std.dev.: 0.00; Training Time: 13.1 ms/epoch
#2 Mean Accuracy (%): 99.99; Std.dev.: 0.01; Training Time: 12.8 ms/epoch
#3 Mean Accuracy (%): 99.99; Std.dev.: 0.01; Training Time: 13.2 ms/epoch
...
#99 Mean Accuracy (%): 99.74; Std.dev.: 0.36; Training Time: 13.0 ms/epoch
#100 Mean Accuracy (%): 99.75; Std.dev.: 0.36; Training Time: 13.0 ms/epoch
```

## Tutorial Slides

You find the slides for this tutorial <a href="https://www.researchgate.net/publication/333648109_Slides_for_the_Convolutional_Tsetlin_Machine_Tutorial_httpsgithubcomcairconvolutional-tsetlin-machine">here</a>.

## Learning Behaviour
<p>
The below figure depicts training and test accuracy for the Convolutional Tsetlin Machine on MNIST, epoch-by-epoch, in a single run.
</p>

![Figure 4](https://github.com/olegranmo/blob/blob/master/performance_by_epoch_MNIST.png)

<p>
Test accuracy peaks at 99.51% after 327 epochs. Further, it climbs quickly in the first epochs, passing 99% already in epoch 2. Training accuracy approaches 100%. 
</p>

<p>
Let us now look at a few examples 10x10 convolution filters produced by the Convolutional Tsetlin Machine for MNIST, including valid convolution positions.  In the figure below, the "*" symbol can either take the value "0" or "1". The remaining bit values require strict matching.
</p>
<p align="center">
  <img width="75%" src="https://github.com/olegranmo/blob/blob/master/MNIST_Clauses_II.png">
</p>
<p>
Above, "0: UL" means "upper left" of the image for digit "0". We can see clearly that "0: UL", "0: UR", "0: LL" and "0: LR" jointly construct the shape "0''. The clauses for the other digits behave similarly, and we thus just illustrate selected patch positions for each digit. As seen, the patterns are relatively easy to interpret for humans compared to, e.g., a neural
network. They are also efficient to evaluate for computers, involving only logical operators.
</p>
  
## Requirements

- Python 3.7.x, https://www.python.org/downloads/
- Numpy, http://www.numpy.org/

## Citation

```bash
@article{granmo2019convtsetlin,
  author = {{Granmo}, Ole-Christoffer and {Glimsdal}, Sondre and {Jiao}, Lei and {Goodwin}, Morten and {Omlin}, Christian W. and {Berge}, Geir Thore},
  title = "{The Convolutional Tsetlin Machine}",
  journal={arXiv preprint arXiv:1905.09688}, year={2019}
}
```

## Licence

Copyright (c) 2020 Ole-Christoffer Granmo

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

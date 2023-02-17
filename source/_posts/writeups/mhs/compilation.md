---
title: "MHSCTF 2023: Writeup Compilation"
date: 2023-02-14 11:08:53
categories:
- ctfs
- mhs
tags:
- compilation
description: "From elegant graph theory algorithms to Flask cookie spoofing, these are various writeups for selected challenges from MHSCTF 2023!"
category_column: "mhs"
permalink: ctfs/mhs/compilation/
thumbnail: /asset/banner/banner-ctfs.png
---

### Intro

I was recently invited by the academic team "DN" (the name, surprisingly, has no inappropriate connotation) to compete in Mentor High School's second CTF iteration, [MHSCTF 2023](https://ctftime.org/event/1861). Although the competition ran for 15 days, we maxed out their 11 challenges in **just under 16 hours** (ignoring solve resets) and managed to take first place. These four writeups were part of the verification process, which came with prize-receiving — enjoy!

---

{% challenge %}
title: Matchmaker 🩸
level: h2
solvers:
- flocto --flag
- enscribe
authors: 0xmmalik
genre: prog
points: 9
description: |
    I've just had the most brilliant idea 😮 I want to write a program that takes all the students and how much they like each other to pair them up so I can maximize the total love in the classroom! Of course, when I say "I," I really mean... "you" ;)  
    Notes: **[SEE BELOW]**  
    `nc 0.cloud.chals.io 22304`
{% endchallenge %}

{% warning %}
**Warning**: Discrete math, graph theory, and combinatorial optimization ahead! If you're unfamiliar with the mathematical symbols used in this writeup, reference this dropdown:

<details>
  <summary><b>Complicated Math Symbols</b>:</summary>
    <ul>
        <li>$\sum$ - Summation</li>
        <li>$\in$ - Element of</li>
        <li>$\notin$ - Not an element of</li>
        <li>$\subset$ - Proper subset of</li>
        <li>$\subseteq$ - Subset of</li>
        <li>$\emptyset$ - Empty set</li>
        <li>$\forall$ - For all</li>
        <li>$\exists$ - There exists</li>
        <li>$\nexists$ - There does not exist</li>
        <li>$\in$ - Element of</li>
        <li>$\notin$ - Not an element of</li>
        <li>$\ni$ - Contains as member</li>
        <li>$\not\ni$ - Does not contain as member</li>
        <li>$\setminus$ - Set minus (drop)</li>
        <li>$\oplus$ - Direct sum</li>
        <li>$\cup$ - Union</li>
        <li>$\cap$ - Intersection</li>
        <li>$x'$ - Prime</li>
    </ul>
</details>

{% endwarning %}

Here are the notes the author provided alongside the challenge description, abridged for brevity:

{% box style:"text-align: left;" %}
- The connection times out after 60 seconds, and there will be 3 iterations.
- The input will be given in $N$ lines, where $N$ represents the number of students. The first line represents the zeroth student, the second line represents the first student, and so on ($50 < N < 100$, $N \bmod 2 = 0$).
- Each line of input contains $N - 1$ integers $R$ (ranged $0 < R < 100$, inclusive); $R$ represents a student's rating of another student at that index, repeated for everybody but themselves.
{% endbox %}

I've cut the notes provided in half to make it a bit more digestable. Before we continue, let's see some sample input from the server for context:

{% codeblock lang:py matchmaker.py %}
from pwn import *

p = remote('0.cloud.chals.io', 22304)
print(r.recvuntilS(b'> '))
{% endcodeblock %}

{% ccb html:true terminal:true wrapped:true %}
<span style="color:#F99157">$ </span> python3 matchmaker.py
[<span style="color:#47D4B9"><b>+</b></span>] Opening connection to 0.cloud.chals.io on port 22304: Done
86 60 67 84 44 4 36 59 100 63 51 6 92 66 36 99 3 69 55 11 21 66 66 81 21 63 76 44 4 87 13 67 0 97 28 13 68 96 47 49 0 18 63 26 73 68 13 63 47 61 0 53 74 56 6 12 5 66 54 47 79 81 84 43 19 6 62 52 6 100 86 64 1 4 38 89 93 6 72 93 63 46 90 29 81 89 5 9 77 23 87 94 73
76 0 74 52 56 60 57 78 48 93 85 66 29 70 96 40 76 62 46 66 69 31 99 47 12 42 43 12 47 19 26 8 26 45 29 27 17 14 15 54 57 78 69 73 55 16 88 50 96 97 34 49 78 3 91 53 28 66 28 28 9 38 87 20 66 28 37 38 94 61 96 99 45 39 52 5 27 5 96 41 31 83 86 32 92 35 96 10 2 97 3 19 88
SKIP_LINE(...)
{% endccb %}

We can do a bit of analysis on what we've received so far.

<div style="display: flex;">
  <div style="flex: 1;">
    <p>The first line, <code>86 60 67...</code>, can be translated into:
    <ul>
      <li>Student 0 -&gt; Student 1 = 86</li>
      <li>Student 0 -&gt; Student 2 = 60</li>
      <li>Student 0 -&gt; Student 3 = 67</li>
    </ol>
  </div>
  <div style="flex: 1;">
    <p>Let's do the same thing for the second line, <code>76 0 74...</code>:</p>
    <ul>
      <li>Student 1 -&gt; Student 0 = 76</li>
      <li>Student 1 -&gt; Student 2 = 0</li>
      <li>Student 1 -&gt; Student 3 = 74</li>
    </ol>
  </div>
</div>

Notice how the student will always skip their own index, which aligns with the author's notes detailing how the integers "represent the students ratings of everybody but themselves." Let's continue with the rest of the notes:

{% box style:"text-align: left;" %}
- Determine the pairings that maximize the students' ratings for each other
    - **Example**: If Student 1 -> Student 7 = 98 and Student 7 -> Student 1 = 87, and Students 1 and 7 are paired, the "score" of this pairing would be $98 + 87 = 185$
- The output should list all maximized pairs (including duplicates; order of the pairs does not matter)
    - **Example**: If Student 0 -> Student 3, Student 1 -> Student 2, and Student 4 -> Student 5, the desired output is: `0,3;1,2;2,1;3,0;4,5;5,4`
- The connection will close if the output is incorrect, and reiterate if correct
{% endbox %}

Let's crack on with the actual algorithm which will be used for solving this!

### Graph Theory Fundamentals

This challenge is a classic example of a concept called "maximum weight matching", which is fundamental in **graph theory** and **discrete mathematics**. Although there is a relatively high prerequisite knowledge floor for understanding these concepts, I'll explain them as best as I can.

Firstly, let's define a **graph**. A graph is a set of **vertices** (or nodes), which are connected by **edges**. Not to be confused with the [Cartesian coordinate system](https://en.wikipedia.org/wiki/Cartesian_coordinate_system), graphs are represented by the [ordered pair](https://en.wikipedia.org/wiki/Ordered_pair) $G = (V, E)$ in which $V$ is a [set](https://en.wikipedia.org/wiki/Set_(mathematics)) of vertices and $E$ is a set of edges:

![Graph](/asset/mhs/graph.svg)

{% definition %}
**Definition**: A set of edges $E$ is defined as $E \subset \\{(x, y)\ |\ (x, y) \in V^2 \textrm{ and } x \neq y\\}$. By this definition, $x$ and $y$ are the vertices that are connected to the edge $e$, called the **endpoints**. It can also be said that $e$ is **incident** to $x$ and $y$.
{% enddefinition %}

We can create a **matching** $M \subseteq E$ within the graph $G$, in which $M$ is a collection of edges such that every vertex $v$ of $V$ is incident to at <u>most</u> one edge of $M$ (meaning two edges cannot share the same vertex). When the highest possible **cardinality** (number of matches) of $G$ is reached, the matching is considered **maximum**, and any vertex $v$ not incident to any edge in $M$ is considered **exposed**. 

{% definition %}
**Definition**: Formally, a matching $M$ is said to be **maximum** if for any other matching $M'$, $|M| \geq |M'|$. In other words, $M$ is the largest matching possible.
{% enddefinition %}

For example, the following is a maximum matching performed on the graph above:

![Matching](/asset/mhs/matching.svg)

{% definition %}
**Definition**: Since there is an exposed vertex in this graph (and because the size of $V$ is odd), $G$ is not considered **perfect**. A **perfect maximum matching** occurs when the size of $V$ is even (or $|V|/2 = |E|$) and there are no exposed vertices.
{% enddefinition %}

Now, let's put **weights** into consideration (i.e. the students' ratings). With a **weighted graph** $G = (V, E, w)$, we can attribute a function $w$ that assigns a weight $w(e)$ to each edge $e \in E$ (defining $w : E \rightarrow \mathbb{N}$):

![Weights](/asset/mhs/weights.svg)

The total weight of a matching $M$, written as $w(M)$, can be defined as:

$$
w(M) = \sum_{e \in M}w(e) \\\\
$$

Solving for $w(M)$ in the example graph above:

$$
w(M) = w((1, 2)) + w((3, 4)) + w((5, 6)) + w((8, 9)) \\\\
w(M) = 5 + 2 + 9 + 6 = 22
$$

Our goal is to maximize $w(M)$ — it is *definitely* not maximized above, since $W(M)$ is not at its highest possible value. We will be tackling it with a combination of two different concepts: [Edmonds' blossom algorithm](https://en.wikipedia.org/wiki/Blossom_algorithm), and the [primal-dual method](https://en.wikipedia.org/wiki/Duality_(optimization)). Although the blossom algorithm is typically meant for [maximum cardinality matching](https://en.wikipedia.org/wiki/Maximum_cardinality_matching) (maximizing the size of $M$ itself, and running in $\href{https\://en.wikipedia.org/wiki/Big_O_notation}{\mathcal{O}}(|E||V|^2)$ time), utilizing it as a subroutine alongside the primal-dual method of linear programming creates a [maximum weight matching](https://en.wikipedia.org/wiki/Maximum_weight_matching) (and running in $\mathcal{O}(|V|^3)$ time).

Firstly, let's get started with how it works.

### The Blossom Algorithm

The core idea behind the blossom algorithm itself involves two concepts: **augmenting paths** and **blossoms** (alongside **blossom contraction**).

#### Augmenting Paths

Given a graph $G = (V, E)$, a path $P$ in $G$ can be considered an **alternating path** if edges in the path are alternatively in $M$ and not in $M$. **Augmenting paths** are a type of alternating, odd-length path that starts and ends with two exposed vertices:

![Augmenting Paths](/asset/mhs/augmenting-paths.svg)

As we can see, $P$ is considered an augmenting path because it ends with two exposed vertices. Augmenting paths are so special in that they can **augment** ("to improve", as per the name) the size of $M$ by one edge. To do so, simply swap the edges in $P$ that are <u>in</u> $M$ with the edges that are <u>not in</u> $M$:

![Augmented Path](/asset/mhs/augmented-path.svg)

{% definition %}
**Definition**: A **matching augmentation** along an augmenting path $P$ is a process by which $M$ is replaced by $M'$, defined as such:

$$
M' = M \oplus P = (M \setminus P) \cup (P \setminus M)
$$

This implies that $|M \oplus P| = |M| + 1$.
{% enddefinition %}

The reason why augmenting paths are so fundamental to the blossom algorithm is that it directly computes a maximum matching — we can reiterate the process of finding augmenting paths until no more are available in $G$. This is described well with [Berge's Theorem](https://en.wikipedia.org/wiki/Berge%27s_theorem):

{% theorem %}
**Theorem**: A matching $M$ in a graph $G$ is maximum <u>if and only if</u> there is no $M$-augmenting path in $G$.
{% endtheorem %}

Here is high-level pseudocode which describes this recursive iteration:

{% ccb html:true gutter1:1-9 caption:'Blossom: Finding Maximum Matching' %}
<span style="color:#e9d3b6"><b>procedure</b></span> <span style="color:#f06431">find_maximum_matching</span>(<span style="color:#f79a32">G</span>, <span style="color:#f79a32">M</span>)
    <span style="color:#dc3958">P</span> = <span style="color:#f06431">find_augmenting_path</span>(<span style="color:#f79a32">G</span>, <span style="color:#f79a32">M</span>)
    <span style="color:#e9d3b6"><b>if</b></span>  <span style="color:#dc3958">P</span> == [] <span style="color:#e9d3b6"><b>then</b></span>
        <span style="color:#e9d3b6"><b>return</b></span> M
    <span style="color:#e9d3b6"><b>else</b></span>
        <span style="color:#dc3958">MP</span> = <span style="color:#f06431">augment_matching</span>(<span style="color:#f79a32">M</span>, <span style="color:#dc3958">P</span>)
        <span style="color:#e9d3b6"><b>return</b></span> <span style="color:#f06431">find_maximum_matching</span>(<span style="color:#f79a32">G</span>, <span style="color:#dc3958">MP</span>)
    <span style="color:#e9d3b6"><b>end if</b></span>
<span style="color:#e9d3b6"><b>end procedure</b></span>
{% endccb %}

{% info %}
**Note**: Under the context of the above pseudocode, `find_augmenting_path()` will always start at an exposed vertex, running a [breadth-first search](https://en.wikipedia.org/wiki/Breadth-first_search) to find an augmenting path.
{% endinfo %}

Therefore, the blossom algorithm will always terminate with a maximum matching.

#### Blossoms (and Blossom Contraction)

The second concept that we need to understand are **blossoms**. Given a graph $G$ and a matching $M$, a blossom $B$ is a "cycle" within $G$ which consists of $2k + 1$ edges, of which exactly $k$ belong to $M$ (a blossom with two matches would have $2(2) + 1 = 5$ edges).

Let's say the algorithm created matchings in a graph with a blossom — although it isn't the maximum matching possible, we cannot augment it any further because the technically available augmenting path is longer than the shortest available path, therefore terminating subotimally:

![Blossom](/asset/mhs/blossom.svg)

This is where **blossom contraction** comes in. The idea is to contract the blossom $B$ into a single "super-vertex" $v_B$, and to treat it as a single vertex in the graph:

![Blossom Contraction](/asset/mhs/blossom-contraction.svg)

From there, we can find valid augmenting paths to augment $M$:

![Augmented Blossom](/asset/mhs/augmented-blossom.svg)

Finally, we can expand the blossom $B$ back into its original form, to reveal the maximum matching:

![Expanded Blossom](/asset/mhs/expanded-blossom.svg)

{% theorem %}
**Theorem**: If $G'$ a the graph formed after contraction of a blossom $B$ in $G$, then $G$ has an augmenting path <u>if and only if</u> $G'$ has an augmenting path.  
(This is called the **Blossoms Theorem**, or the **Main Theorem of Botany**)
{% endtheorem %}

With the fundamental concepts of this beautiful algorithm covered, the only thing we need to wrap our heads around is how to adopt this with weighted graphs. I'll be referencing Galil's [Efficient Algorithms for Finding Maximum Matching in Graphs](http://www.cs.kent.edu/~dragan/GraphAn/p23-galil.pdf) for this section.

### Primal-Dual Method

So... the primal-dual method. I've been personally studying this topic (alongside linear programming in general, from which this method is derived from) for the past couple days, and let's just say that I absolutely have no clue what is going on:

![Scary Math](/asset/mhs/scary-math.png)

So I asked ChatGPT-3 for a bit of help. Hopefully this will help you too:

<div class="flex-container">
  <div style="width: 70px">
    <img style="border-radius: 16px;" src="https://www.gravatar.com/avatar/bac00b54948ebf5ea072824aeafa890f?d=identicon&s=70">
  </div>
  <div>
    <div class="box" style="text-align: left;">
      Explain the primal-dual method to someone completely new to algorithms and graph theory.
    </div>
  </div>
</div>

<div class="flex-container">
  <div style="width: 70px">
    <img src="/asset/mhs/chatgpt-icon.png">
  </div>
  <div>
    <div class="box" style="background-color: #064133; text-align: left;">
"The primal problem is the original problem that we want to solve, and the dual problem is a related problem that helps us solve the primal problem. In graph theory, the primal problem might be to find the shortest path between two nodes in a graph, while the dual problem might be to find the minimum cost to connect all the nodes in the graph.

The method works by starting with a feasible solution to the dual problem and iteratively improving it until it becomes the optimal solution. At each step, the method adjusts the primal and dual variables to satisfy a set of complementary slackness conditions, which ensure that the solution is both feasible and optimal."
    </div>
  </div>
</div>

<div class="flex-container">
  <div style="width: 70px">
    <img style="border-radius: 16px;" src="https://www.gravatar.com/avatar/bac00b54948ebf5ea072824aeafa890f?d=identicon&s=70">
  </div>
  <div>
    <div class="box" style="text-align: left;">
      Can you explain what slack is in linear programming (under the context of maximum weighted matching)?
    </div>
  </div>
</div>

<div class="flex-container">
  <div style="width: 70px">
    <img src="/asset/mhs/chatgpt-icon.png">
  </div>
  <div>
    <div class="box" style="background-color: #064133; text-align: left;">
"In the context of linear programming for maximum weighted matching, slack is defined as the amount by which the current matching fails to satisfy the optimality conditions for a maximum weight matching. More formally, for a graph $G=(V,E)$ and a matching $M$, the slack of an edge $e$ is defined as:

$$s_e= \begin{cases}w(e)-u(x)-v(y) & \text { if } e \notin M \\\\ u(x)+v(y)-w(e) & \text { if } e \in M\end{cases}$$

where $x$ and $y$ are the endpoints of $e$, $w(e)$ is the weight of $e$, and $u$ and $v$ are dual variables associated with the vertices of $G$. The dual variables are used to derive bounds on the weights of the edges, and are updated during the execution of the primal-dual algorithm.<br><br>
Intuitively, the slack of an edge measures the amount by which the edge can be "improved" in order to increase the total weight of the matching. A positive slack indicates that the edge can be added to the matching to increase its weight, while a negative slack indicates that the edge is over-saturated and should be removed from the matching. The slack values are used to identify augmenting paths in the graph, which can be used to improve the current matching until it satisfies the optimality conditions for a maximum weight matching."
    </div>
  </div>
</div>

### "Borrowing" an Implementation

Yes, the blossom algorithm is the most elegant thing I've ever seen. No, I am absolutely never going to implement the blossom algorithm from scratch. I'm not even going to try. Yes, it's super fun to visualize and understand the concepts, but could you even imagine trying to implement this in code, and deal with "neighbors" and "forests" and "visited nodes?" I can't.
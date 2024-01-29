---
title: An Interview Problem
tags: interview-question dynamic-programming chess
mathjax: true
excerpt: "Solving an interview problem that intrigued me."
published: true
---

I encountered this question during an interview for a popular software company.
I wasn't able to reach a satisfactory solution to it during the course of the
interview (and although I cleared that round, I didn't get the role). However, I
felt sufficiently close to the solution that I was tempted to work on it after
the interview was over.

## Problem Statement

You are given chess board (of arbitrary size $M \times N$) where each square has
a certain non-negative value. A single rook is placed on the board. The rook can
move horizontally or vertically as in regular chess. However, it can only move
in this fashion to squares that have a strictly lower value than the square it
starts from. Given the rook's starting square, find the length of the longest
path it can take on the board.

Here's a sample board of size $6 \times 5$ --

|2|3|1|4|5|
|:-:|:-:|:-:|:-:|:-:|
|6| 7| 9| 2| 1|
|3| 5| 7| 1| 9|
|8| 8| 8| 7| 6|
|9| 4| 2| 5| 8|
|2| 1| 4| 7| 9|

Let's say the rook is located at coordinates $(3, 0)$.

|2|3|1|4|5|
|:-:|:-:|:-:|:-:|:-:|
|6| 7| 9| 2| 1|
|3| 5| 7| 1| 9|
|♖| 8| 8| 7| 6|
|9| 4| 2| 5| 8|
|2| 1| 4| 7| 9|

Given its starting square is of value $8$, it can only move (vertically or
horizontally) to squares that are of value strictly lower than $8$.

|✅|3|1|4|5|
|:-:|:-:|:-:|:-:|:-:|
|✅| 7| 9| 2| 1|
|✅| 5| 7| 1| 9|
|♖| ❌| ❌| ✅| ✅|
|❌| 4| 2| 5| 8|
|✅| 1| 4| 7| 9|

The objective is to find the *longest* path the rook can take on the board. It
suffices to simply return the length.

## Approach

This immediately felt like a dynamic programming problem. From a given state (of
where the rook is located), you can call an imaginary routine
$\text{longestPathFrom(rook)}$ to find the length of the longest path from that
state. This is independent of how you arrived at that state. Thus, we can
formulate a preliminary recurrence relation as --

$$ \text{longestPathFrom(state)} = 1 + \text{longestPathFrom}(\text{state}') $$

for all states that are *immediately reachable* from the current state. Given
how the rook moves, these immediately reachable squares are those on the same
rank or file as the rook could be moved to in a single move (provided their
value was strictly less than the value of the rook's square).

However, there may be multiple ways of reaching the same square, and some may be
longer than others. Hence, we need to modify the recurrence relation to --

$$ \text{longestPathFrom(state)} = \max_{\text{state}'\ \in \text{
    reachable}}\left(0, 1 + \text{longestPathFrom}(\text{state}')\right) $$

## Solution

A simple algorithm to implement this is --

> $M$: the number of rows in the board   
> $N$: the number of columns in the board  
> $\text{values}[M][N]$: the values of each cell in the board that govern
> reachability  
> $(i, j)$: the (row, column) of the cell *from* which we want to find the
> longest path  
> Initialize $\text{dp}[i][j] \leftarrow \varnothing, \forall i \in [1, M], j \in [1, N]$  
>
> $\text{longestPathFrom}(i, j)$  
> $\quad$**if** $\text{dp}[i][j] == \varnothing$ **then**  
> $\qquad d_{\max} \leftarrow \max_{i' \in [1, M]}(0, 1 +
> \text{longestPathFrom}(i', j))$  
> $\qquad d_{\max} \leftarrow \max_{j' \in [1, N]}
> (0, 1 + \text{longestPathFrom}(i, j'))$  
> $\qquad \text{dp}[i][j] \leftarrow d_{\max}$  
> $\quad$**endif**  
> **return** $\text{dp}[i][j]$

Here's the algorithm implemented as a full solution in Python --

{% highlight python linenos %}
def print_matrix(matrix):
    print('\n'.join(['\t'.join([str(cell) for cell in row]) for row in matrix]))

def max_dist(values, to_i, to_j, dp):
    """
    max_dist(i, j) returns the max length path from coordinates (i, j)
    """
    if dp[to_i][to_j] is None:
        curr_max_dist = 0 # default unexplored max distance (no reachable squares)
        for k in range(len(values[0])):
            if values[to_i][k] < values[to_i][to_j]:
                curr_max_dist = max(curr_max_dist, 1 + max_dist(values, to_i, k, dp))
        for k in range(len(values)):
            if values[k][to_j] < values[to_i][to_j]:
                curr_max_dist = max(curr_max_dist, 1 + max_dist(values, k, to_j, dp))
        dp[to_i][to_j] = curr_max_dist
    return dp[to_i][to_j]

if __name__ == '__main__':
    with open('input.txt', 'r') as inputfile:
        n, m = map(int, inputfile.readline().split())
        values = [[] for _ in range(n)]
        for i in range(n):
            values[i] = list(map(int, inputfile.readline().split()))
        u, v = map(int, inputfile.readline().split())
        dp = [[None] * m for _ in range(n)]
        ans = max_dist(values, u, v, dp)
        # print_matrix(dp)
        print(ans)
{% endhighlight %}
Here are the contents of a sample `input.txt` file you can use --

```
6 5
2 3 1 4 5
6 7 9 2 1
3 5 7 1 9
8 8 8 7 6
9 4 2 5 8
2 1 4 7 9
3 0
```

The correct answer is $7$. Here's another one

```
6 5
4 4 4 4 4
5 5 5 5 5
6 6 6 6 5
7 7 7 6 5
8 8 7 6 5
9 8 7 6 5
5 0
```

The correct answer is $5$.
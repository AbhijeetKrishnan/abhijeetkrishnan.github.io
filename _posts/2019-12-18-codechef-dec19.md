---
title: Codechef December 2019 Long Contest (Div. 1)
tags: competitive-programming codechef dec19a
mathjax: true
excerpt: "An editorial for the Dec '19 Div. 1 long contest on Codechef."
---

## [BINXOR - Binary XOR](https://www.codechef.com/DEC19A/problems/BINXOR/)

This stumped me for a good bit of time. I tried to think of a variety of approaches, but nothing panned out. Thankfully, they helped me build some intuition about the problem which helped me formalize the final solution.

Since the strings $A$ and $B$ can be reordered arbitrarily, we can represent them using the count of $0$'s and $1$'s in them. Since their length is given as $n$, we can represent them instead in terms of just the count of $0$'s (the count of $1$'s will be $n - cnt_0$). The strings now become $cnt_{0,A}$ and $cnt_{0,B}$.

Let's say we know $cnt_{0, A \oplus B}$. How many binary numbers could we form? Since we have $cnt_{0, A \oplus B}$ $0$'s and $n - cnt_{0, A \oplus B}$ $1$'s, the total number of combinations is $^nC_{cnt_{0, A \oplus B}}$.

How does this help us? Let us try to find which values of $cnt_{0, A \oplus B}$ are possible, given some $cnt_{0,A}$ and $cnt_{0,B}$.

[AC Solution](https://www.codechef.com/viewsolution/28187186)

## [BINADD - Addition](https://www.codechef.com/DEC19A/problems/BINADD)

This problem becomes easy once you make the connection between the addition algorithm presented in the problem and the full-adder addition algorithm. I did not see this myself at first, and instead searched for 'addition using bitwise operations' and found a relevant [GeeksForGeeks article](https://www.geeksforgeeks.org/add-two-numbers-without-using-arithmetic-operators/). The connection is that the variables $U$ and $V$ represent the sum and carry respectively. With a little manual addition on paper, it is easy to see when the loop is repeated - when you have a _chain_ of carries. A _chain_ of carries begins when there is a $1 - 1$ pair (the bits at some position in $A$ and $B$ are both set). It continues when only one bit in either $A$ or $B$ is set, and ends when this condition is not true.

We simply iterate through the bit strings backwards i.e. from LSB to MSB (we can pad the shorter string with zeroes to simplify iteration). We check the the bit pairs to find out if we are in a carry chain or not. We maintain the length of the longest carry chain found, and return it as the answer after traversing the bit strings.

The time complexity should be $O(T \cdot N)$.

[AC Solution](https://www.codechef.com/viewsolution/28215243)

## [CHFRAN - Chefina and Ranges](https://www.codechef.com/DEC19A/problems/CHFRAN)

This problem again made me think of a number of different approaches such as DSU and minimum vertex connectivity of a graph. Finally, the solution boiled down to the familiar technique of finding the number of _active_ ranges at any point. If we know this, we can choose to delete all active ranges at some point to create a "gap" in the ranges, which means we can put all ranges to the left of the gap in one set, and the ranges to the right of the gap in the other.

We do this by entering all start and end points into a single list, with some additional mechanism to differentiate between a start point and an end point (I turned each point into a pair with the second element representing whether it was a start point or end point). Start points must be placed before end points of the same value. We then iterate through this list of points, while maintaining a count of active ranges at some point. If we encounter a start point, we have entered a range and increment our count of active ranges. If we encounter our end point, we have exited a range and decrement our count of active ranges. We can now decide if we want to remove all active ranges, and thus update a variable containing the minimum number of active ranges at a point. Finally, we return this minimum.

The time complexity should be $O(T \cdot N \log n)$.

[AC Solution](https://www.codechef.com/viewsolution/28296886)

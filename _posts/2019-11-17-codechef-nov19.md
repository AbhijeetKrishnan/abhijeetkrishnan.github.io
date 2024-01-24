---
title: Codechef November 2019 Long Contest (Div. 2)
tags: competitive-programming codechef nov19b
mathjax: true
excerpt: "An editorial for the Nov '19 Div. 2 long contest on Codechef."
---

## [SC31 - Weapon Value](https://www.codechef.com/NOV19B/problems/SC31/)

This was a fairly straightforward implementation based question. It would have taken longer to read and understand the problem statement than to come up with or even code the solution.

It is easy to see from the question description that for any battle between contestants $i$ and $j$ with weapon strings $S_i$ and $S_j$, the weapon string of the victor is given by $S_i \oplus S_j$, where $\oplus$ denotes the logical XOR operation. Given that the XOR operation is associative and commutative, we can ignore any specific orders of battles and simply XOR all the weapon strings together. 

The complexity should be $O(T \cdot N)$.

[AC Solution](https://www.codechef.com/viewsolution/27739777)

## [HRDSEQ - Hard Sequence](https://www.codechef.com/NOV19B/problems/HRDSEQ)

The constraints here are terribly small and it is easy to implement the rules for the sequence directly. We can generate the sequence offline for $128$ terms in $O(N^2)$, and then compute the answer for each query in $O(N)$, resulting in a total complexity of $O(N^2 + T \cdot N)$.

There is some optimization potential here though. Generating the sequence can be optimized to $O(N)$ by keeping track of the position of when a particular number in the sequence was last seen. Concretely, when calculating term $x_i$ in the sequence, we query a data structure for the position where $x_{i-1}$ was last seen, say $j$. $x_i$ is then simply $i - 1 - j$ if there exists such a position, else it is $0$.

[AC Solution](https://www.codechef.com/viewsolution/27739839)

Any sequence you find should typically be plugged into [OEIS](https://oeis.org/) to learn more about it. The sequence in this problem is the [Van Eck's sequence](https://oeis.org/A181391) and it has a nice [video](https://www.youtube.com/watch?v=etMJxB-igrc) from [Numberphile](https://www.youtube.com/channel/UCoxcjq-8xIDTYp3uz647V5A) on it. Unfortunately, neither the OEIS page nor the video offer any insight into how we might solve the problem for larger constraints, say $N = 10^9$, as asked in a comment [here](https://discuss.codechef.com/t/hrdseq-editorial/43707/3).

## [PHCUL - Physical Exercise](https://www.codechef.com/NOV19B/status/PHCUL,rashomon)

This problem stumped me for a good amount of time until I realized that I had misread the constraints and was in fact trying to solve a much tougher problem than was asked.

A naive solution is to simply check every triplet of points and find the minimum distance. This would run in $O(T \cdot N \cdot M \cdot K \cdot \lg(MAX))$ and would only pass the first subtask. The $\lg(MAX)$ factor comes from calculating the square root (in the distance calculation). The maximum value of the squared distance (based on the constraints) can be $2 \times 10^{18}$.

I missed the constraint that the sum of $N + M + K$ over all test cases does not exceed $15,000$ and was struggling to come up with a greedy solution. A greedy solution would try to find the closest point from the start in set $N$ or $M$, then find the closest point in the other set, and finally find the closest point in set $K$. I believed that some clever way of precomputing the distances would allow me to implement this solution in $O(T \cdot (N + M + K) \cdot \lg(MAX))$.

However, a greedy solution does not work for this problem. Consider the test case in [this](https://discuss.codechef.com/t/phcul-editorial/43712/20) comment. The greedy solution would choose the point $(9, 10)$ in the start, leading to a worse solution than the optimal first choice of $(11, 10)$.

In light of this, as well as the constraints, it is clear that an $O(T \cdot N \cdot M \cdot \lg(MAX))$ solution would also work. We just need to pre-compute some distances to reduce the complexity from cubic to quadratic. We can do this by finding the closest point in set $K$ for each point in sets $N$ and $M$, and iterating over every possible pair of points in sets $N$ and $M$. The final path for some ordered pair $(n, m)$ would be $start \rightarrow n \rightarrow m \rightarrow closest[m] \rightarrow finish$. 

This solution runs in $O(T \cdot (N \times M + (N + M) \times K) \cdot \lg(MAX))$.

[AC Solution](https://www.codechef.com/viewsolution/27740853)

## [CAMC - Chef and Minimum Colouring](https://www.codechef.com/NOV19B/problems/CAMC)

This was a nice problem. It seemed complex at the start, but I boiled it down to a form I knew there must be a neat solution for, and somewhat shamefully, found that solution online.

The boxes form $M$ groups based on their index. Box $A_i$ will get placed into group $i \% M$. Now, we need to choose one box from each group such that the difference between the maximum and minimum number of balls in a box is minimized.

I tried to solve this problem for a while without success. I searched online and found [this](https://www.geeksforgeeks.org/find-smallest-range-containing-elements-from-k-lists/) GeeksforGeeks article describing exactly how to solve the problem using a min-heap. The idea is to use the min-heap to maintain a window of $M$ elements. We advance the window of $M$ elements by retrieving the minimum value element, which is compared against the maximum value element in the window. The maximum value in the window can be maintained separately, or by using a [min-max heap](https://people.scs.carleton.ca/~santoro/Reports/MinMaxHeap.pdf) (see [this](https://www.codechef.com/viewsolution/27819250) AC solution which uses a min-max heap). The retrieved element is then replaced by the next element in that list. 

The time complexity should be $O(T \cdot ((M \cdot \left\lfloor{\frac{N}{M}}\right\rfloor \cdot (\lg(\left\lfloor{\frac{N}{M}}\right\rfloor) + \lg(M)))$.

[AC Solution](https://www.codechef.com/viewsolution/27776080)

The [official editorial](https://discuss.codechef.com/problems/CAMC) uses a two-pointer technique to maintain the sliding window, and checks to see when we have found boxes from all $M$ colours. The complexity provided is incorrect however, since the solution requires sorting.

I also benefited a lot in this problem from following the testing protocol described by [Errichto](https://www.youtube.com/channel/UCBr_Fu6q9iHYQCh13jmpbrg) in [this](https://www.youtube.com/watch?v=JXTVOyQpSGM) video.

## [WEIRDO - From Zero to Infinity](https://www.codechef.com/NOV19B/problems/WEIRDO)

This was an easy problem masquerading as a tricky one. I overlooked an elementary error in an earlier part of my solution and instead got bogged down in trying to correct a later part (which was actually correct).

We first need to identify which recipes belong to Alice, and which to Bob. The condition for a recipe to belong to Alice states that it must not have any substring containing more consonants than vowels. We can also deduce that if a particular substring violates this condition, then all substrings containing that substring also violate that condition. Thus, we can look for the smallest possible substring violating this condition. If we find one, we label the recipe as belonging to Bob, else it belongs to Alice. We can check all possible substrings of length $3$, and check whether this condition holds. This can be done in $O(3 \cdot \|S\|)$ per substring. Some tricky cases are strings like "bab, bb, aababa", which belong to Bob.

Given accurate labels for all the recipes, we can calculate the statistics $x_c$ and $f_{x_c}$  $\forall c$ straightforwardly. We cannot calculate the score directly since the numbers involved are very large ($\approx 10^{10^5}$). To simplify this, we can notice that all operations involve multiplication and division and thus we can reduce them to additions and subtractions using logarithms.

The time complexity is $O(\sum\|S\| + \sum\|L\| \cdot \lg(MAX))$.

[AC Solution](https://www.codechef.com/viewsolution/27793523)

The alternate approach described in the [official editorial](https://discuss.codechef.com/t/weirdo-editorial/43717) is a clever way to multiply/divide the numbers together while ensuring the resulting value does not get too big. This is possible since there are upper and lower bounds provided on the final value.

## [LSTBTF - Smallest Beautiful Number](https://www.codechef.com/NOV19B/problems/LSTBTF/)

This was a fun problem which engaged me a lot.

My approach differs from the editorial by quite a bit. It was similar to [this](https://discuss.codechef.com/t/lstbtf-editorial/43793/18) comment I found on the editorial page. Starting from a number consisting of all $1$'s, we can observe that by changing a $1$ to a $2$, we increase the sum of digits by $3$. Similarly, we calculate the increase by changing a $1$ to each number from $3$ to $9$. Using the smallest possible number of increases, we need to raise the sum of digits of a number from $N$ to a perfect square.

I simply precalculated all possible increases using $i$ digit changes in a dictionary. If a key is present in the $i$-th index, it denotes that the key value is achievable using $i$ digit changes, with the value of that key containing the actual digits we must change the number to. I precalculated up to $40$ digit changes, which are enough to solve the problem. The maximum value of digit changes required is apparently $27$ as stated in the editorial, however I don't know how we would arrive at that number in a reasonable amount of time.

Now, I simply traversed all square numbers from $N$ (the initial sum) to $81 \times N$ (the maximum change possible as a result of changing all digits to $9$), and checked whether that square number was achievable in some number of digits. I returned the number corresponding to the smallest change. 

The time complexity of the initial pre-computation step is difficult to characterize since in the worst case it is $O(9^{40})$, but in practice, many values overlap and it runs significantly faster than that. The time complexity for the testing phase is $O(T \cdot 27 \cdot \sqrt{N} \cdot 9^{27})$. The $9^{27}$ factor is considering the maximum number of values at a particular level of the dictionary table, which again in practice is much smaller due to overlap. 

[AC Solution](https://www.codechef.com/viewsolution/27779069)

I haven't yet analyzed the [official editorial](https://discuss.codechef.com/t/lstbtf-editorial/43793).

## [FAILURE - Single Point of Failure](https://www.codechef.com/NOV19B/problems/FAILURE)

This problem greatly consumed me. My final solution was not optimal, and only passed due to some hacky optimizations. Definitely a good opportunity to upsolve.

The idea is to find a node in an undirected graph, which upon removal will turn the graph into a tree. If such a point exists, then the least numbered point is to be returned. If such a point does not exist, or if the graph is already a tree, then we must return $-1$.

Cycle detection is a easily solved on an undirected graph using DFS. We can also identify the cycles on which a vertex lies using a modification of DFS which marks vertices upto their LCA if a back-edge is detected (see [this](https://discuss.codechef.com/t/loncyc-editorial/19701) editorial for details). I was trying to use this technique to identify which cycle every graph lay in, and delete the least numbered vertex which lay in the most cycles.

However, this process did not work because the cycles identified were overlapping. This caused the incorrect failure node to be identified. I did not find a solution for this and simply optimized my naive solution of removing every node which lay in the maximum number of cycles and checking the remaining graph. This somehow passed.

The time complexity of my sub-optimal solution is $O(T \cdot N \cdot (N + M))$.

[AC Solution](https://www.codechef.com/viewsolution/27840508)

The [official editorial](https://discuss.codechef.com/t/failure-editorial/43792) solves this by identifying conditions on a vertex for it to be a failure node. The conditions themselves can be checked using a DFS traversal. The first condition is straighforward. The second condition corresponds to my observation that a failure node must lie on a non-overlapping cycle.

A much more intuitive solution is given in [this](https://discuss.codechef.com/t/failure-editorial/43792/2) comment. We first remove all bridges, which can be found using a DFS in $O(N + M)$. Then we remove the lowest numbered vertex with the most edges and test the remaining graph for cycles.

## [TRICOL - (Challenge) Coloring Triangulations](https://www.codechef.com/NOV19B/problems/TRICOL)

Unfortunately, I did not submit a solution to this problem during the contest. I will upsolve it soon.

# Overall Impressions

The problems definitely felt easier this contest, as clearly evidenced by my performance. All problems used fairly basic concepts as well. I made some errors in thinking - not reading the constraints, overlooking simple conditions etc., but I am pleased with my performance. My promotion into Div. 1 should make things more challenging now, however.
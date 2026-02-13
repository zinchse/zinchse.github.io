---
layout: post
title: Leetcode Marathon (**do not repeat!**)
date: 2023-08-12 14:59:00
description: an attempt to solve leetcodes's problems 100 days in a row
disqus_comments: true
related_posts: false
img: /assets/img/leetcode_challenge_post/leetcode_challenge.jpg
---

<div style="text-align:center;">
    <img src="/assets/img/leetcode_challenge_post/leetcode_challenge.jpg" alt="contest logo" width="450px">
</div>

 <br>

I was going to try to get in the **habit** of *solving the `leetcode` problem before breakfast*. And do it 100 times in a row to start with.

On about day 42-45 of my venture, **I stopped seeing any progress**. I learned to test my code. I learned to estimate asymptotics by constraints. I learned to find the right patterns. And the whole idea started to seem like a **waste of time**. I decided to see it through to the end and spent the remaining 60 days solving problems with almost no interest or excitement. I would give this idea a 4/10 rating now (*and that's why I don't recommend repeating*). But the first 40 days were really **awesome**. 

Here's what new things I got out of it.

###  `Leetcode` tasks are best not solved on the `leetcode` platform. I'm serious.



That `run` / `submit` button completely shuts off the desire to think. The best I've come up with is to **copy** the problem description to an external editor (`vim` / `notepad` / `jupyter`, etc.) and **write** the solution there. Moreover, when the solution was written, it need to be **tested**. This approach slows down the process, but the % of successful submits *goes into space*.

### There are not so many patterns for solving problems. Really. 

I have almost **stopped seeing something new** in problem solving. Now I have solved about 380 tasks in total, and the solution of almost any task looks the same:
    
1. look at the constraints of the problem, this gives an understanding of what asymptotics is expected from the solution
    
2. find the pattern that is expected to be used in solving the problem, patterns can be `graph traversals`, `binary search`, `dynamic programming` and so on ...

3. double-check that the patterns I have found are supposed to be used, this is easy enough to do by introducing invariants whose maintenance is guaranteed by the patterns I have chosen

For an introduction to patterns, I suggest the following [[resource]](https://mm1705.github.io/leetcode-patterns/)

### Tips

Besides basic things like compact graph traversal, all kinds of binary search (yes, it comes in left, right, and classical), I've picked up a couple of nice snippets:

-   `for el in itertools.chain(array, [TRIGGER])` 
    
    Sometimes when processing an `array` it is convenient to rely on the triggering of some event (*e.g., the change of a unique character in the example below*). This simplifies life, but requires additional logic in situations where the event does not occur (*e.g., for the last characters in the array*). This situation can be avoided by independently triggering the event at the right moment (*e.g., by expanding the initial array with a trigger symbol*).

    ```python
    def max_product(s: str) -> int:
        """
        returns the maximum product 
        of the lengths of two adjacent subarrays 
        containing only one unique character (per subarray)
        """
        TRIGGER = "$"
        max_product = prev_cnt = cur_cnt = 0
        prev_char, cur_char = "$", s[0]
        for char in itertools.chain(s, [TRIGGER]):
            if char == cur_char:
                cur_cnt += 1
            else:
                max_product = max(max_product, cur_cnt * prev_cnt)
                prev_char, cur_char = cur_char, char
                prev_cnt, cur_cnt = cur_cnt, 1
        return max_product
    ```
    
    The example above is toy-like but demonstrative. In it, we can avoid adding a trigger, but then we would have to do an additional comparison at each iteration.
    

-   `get_ngbs`, `for dx, dy in [(-1, 0), ...]` 

    When working with graphs (especially if they are represented by a chessboard or something similar), it is convenient to introduce a function that returns a list of neighbors.

    ```python
    def get_ngbs(x: int, y: int) -> List[Tuple[int, int]]:
        res = []
        for ngb in get_possible_ngbs(x, y):
            if condition(x, y, ngb):
                res.append(ngb)
        return res
    ```

    The `condition` above can be, for example, the absence of obstacles or the freedom of the cell.  
    And the possible neighbors are taken by iterating over all possible directions of movement. 

    ```python
    # four-way movements around the current node
    def get_possible_ngbs(x: int, y: int) -> List[Tuple[int, int]]:
        res = []
        for dx, dy in [(-1, 0), (0, -1), (1, 0), (0, 1)]:
            if 0 <= x + dx < max_x and 0 <= y + dy < max_y:
                res.append((x + dx, y + dy))
        return res
    ```

-   `@lru_cache`
    
    When working with dynamic programming, where the order of element traversal is not obvious, you can put all the work on caching and recursion by using the `@lru_cache` decorator.

    ```python
    @lru_cache(None)
    def dp(el: 'Element') -> None:
        # calculating dp equation by using `dp` values
        pass

    for el in els:
        dp(el)

    return dp[TARGET_EL]
    ```


-   `defaultdict(lambda: DUMMY_VALUE)`

    Also in dynamic programming it is sometimes useful to set some default `DUMMY_VALUE`. For example, if recursive calculations are looking for the minimum of something, we can use `DUMMY_VALUE = float('inf')`. This will avoid checking that the element whose value we are accessing has already been retrieved at some point.


### Top problems

-   **[[substring-with-largest-variance]](https://leetcode.com/problems/substring-with-largest-variance/description/)**: just a nasty task due to the many different cases

-   **[[minimum-speed-to-arrive-on-time]](https://leetcode.com/problems/minimum-speed-to-arrive-on-time/description/)**: interesting edge cases

-   **[[longest-subarray-of-1s-after-deleting-one-element]](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/description/)**: interesting edge cases

-   **[[find-k-pairs-with-smallest-sums]](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/description/)**: interesting combination of heap and set

-   **[[insert-delete-getrandom-o1]](https://leetcode.com/problems/insert-delete-getrandom-o1/description/)**: interesting combination of array and hash

-   **[[lru-cache]](https://leetcode.com/problems/lru-cache/)**: interesting combination of a hash table and a list; trick of introducing front / back nodes

-   **[[maximum-running-time-of-n-computers]](https://leetcode.com/problems/maximum-running-time-of-n-computers/description/)**: non-obvious strategy without a short proof of correctness

-   **[[shortest-path-to-get-all-keys]](https://leetcode.com/problems/shortest-path-to-get-all-keys/description/)**: the most complicated bfs I've ever seen

-   **[[tallest-billboard]](https://leetcode.com/problems/tallest-billboard/description/)**: beautiful reduction of the complexity

-   **[[check-if-it-is-a-straight-line]](https://leetcode.com/problems/check-if-it-is-a-straight-line/description)**: beautiful application of math (vector product)

-   **[[stone-game-ii]](https://leetcode.com/problems/stone-game-ii/description/)**: interesting property of the zero-sum games

-   **[[power-of-heroes]](https://leetcode.com/contest/biweekly-contest-104/problems/power-of-heroes/)**: good properties of functionals (linearity)

-   **[[intersection-of-two-linked-lists]](https://leetcode.com/problems/intersection-of-two-linked-lists/)**: small (but important!) step -- finding the nearest common node of two lists

-   **[[number-of-subsequences-that-satisfy-the-given-sum-condition]](https://leetcode.com/problems/number-of-subsequences-that-satisfy-the-given-sum-condition/description/)**: trick to represent the solution as a binary string
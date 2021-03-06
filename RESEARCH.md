# Research

#### Data Structures

After extensively testing golang's built-in linked lists library, `container/list`, raw linked lists (smaller feature set), skip lists, btrees, and b+trees, we display some impressive results:

![speed test](/assets/unknown-3.png)

_Fig. 1 - built-in linked lists vs. raw linked lists vs. skip list vs. btree vs. b+tree operations (100 million records)_

As seen in _Fig. 1_, the speed advantages of b+trees for our use case are enormous. With `100 Million Records`, we get a random search time of only ~6 microseconds using a b+tree.

This is the major advantage that b+trees present, as a significantly large majority of operations will be search, as opposed to data modification operations.

There is a clear slowdown when it comes to modifying the data in the tree, but by sacrificing time on the speed of modification, we by far make that time up in search performance.

With this performance, we can recognize for about `170 thousand` searches per second.

Comparing that to a b+tree written in Node.JS, we see the speed advantages of Go as well:

![speed comparison](/assets/unknown-1.png)

_Fig. 2 - b+tree in Node.JS (100 million records)_

At `141 microseconds`, golang is about 23x faster than Node.JS, not even counting that it is more memory efficient as well.

**This all became useless when we tested raw maps:**

Using the same data set, we saw the following results using a map:

![big data map](/assets/unknown-6.png)

_Fig. 3 - Map Performance (100 million records)_

While it did take a while to construct, the performance cannot be ignored. Running consistently between 1.1 to 1.7 microseconds is significantly better than 6 to 8 microseconds.

Furthermore, with maps, we build a map for each field in the JSON object. This means we have the ability to filter which fields are searched ("SPECIFIC SEARCH" as seen in Fig. 3), enabling us to search even faster.

**We thought searching multiple maps concurrently would yield even better performance, we were wrong:**

Using `goroutines`, golang's version of threading, we observed the following performance metrics on identical data sets:

![concurrent goroutine search](/assets/unknown-5.png)

_Fig. 4 - Map concurrent goroutine search (100,000 records)_

![sync search](/assets/unknown-4.png)

_Fig. 5 - Synchronous search (100,000 records)_

This test, on a dataset with 10 fields (10 maps), resulted in significantly slower performance when using `goroutines`. Seemingly counter intuitive, but makes development easier as we don't have to deal with the complexity of concurrency. We suspect this is due to the concurrency running on only a single core, and thus wasting time switching between threads.

_Also notice how similar the times of 100 million and 100,000 records are so similar? O(n) performance baby!_

**Just when we thought we had the solution, our scope invalidated it:**

Not only are maps incredibly memory inefficient (so many duplicate characters and element), but we wanted to add features like prefix searching, suffix searching, and fuzzy searching. In order to do this, we leveraged the power of `radix trees`.

`Radix trees` are an extension of `Tries`, which are string key prefix trees (see [trie](https://en.wikipedia.org/wiki/Trie) and [radix tree](https://en.wikipedia.org/wiki/Radix_tree)). This allows us to have a highly memory efficient data structure that allows us to perform prefix, suffix, and fuzzy searches based on string input.

`Radix trees` are optimized versions of `Tries`, as proven in testing below:

![trie test](/assets/unknown-8.png)

_Fig. 6 - Trie test_

![radix tree test](/assets/unknown-7.png)

_Fig. 7 - Radix Tree test_

Running the exact same tests, we can see that radix trees perform better in every category.

This allows us to do the following searches:

_PSUEDO CODE:_
```
INSERT fire
ok

INSERT firebase
ok

INSERT firetruck
ok

FIND fire
fire
# in max O(m) time, where m is the number of characters in the search

PREFIX_SEARCH fire
fire firetruck firebase
# in max O(p) + O(m) time, where p is the number of children under the search term

FUZZY_SEARCH ftuc
firetruck
# in max O(p*m) time, where p and m are same as above
```

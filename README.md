
# The igraph library

Welcome to CSCLAB's fork of igraph C core.

This repository contains a patched version multilevel function (a.k.a. the fast unfolding algorithm). The function is speeding up by removing all the quicksorts called. By doing so, the multilevel function is now a near linear time algorithm. Due to the different iteration order and the id relabeling order, ties will be broken in a different way. Hence, the behavior of our fast unfolding algorithm is not strictly the same as the function in the original igraph, e.g., the result of **test 177** in `make check` of our algorithm is different from the result of the original one.

## Performance Comparison
The time complexity of the multilevel function is improved from **O(k⋅m⋅log m)** to **O(k⋅m)**, where m is the number of edges and k is the depth of recursive calls (which is supposed to be in the order of log(m) ).
- benchmark code

  The following code is compiled with parameters `-std=c++14` and `-O3`
  ```cpp
  #include <chrono>
  #include <igraph.h>
  #include <iostream>
  using namespace std;
  using namespace chrono;

  int main() {
      igraph_t g;
      igraph_erdos_renyi_game(&g, IGRAPH_ERDOS_RENYI_GNP, 2000, 0.5,
                              IGRAPH_UNDIRECTED, IGRAPH_NO_LOOPS);
      igraph_vector_t modularity, membership, edges;
      igraph_matrix_t memberships;
      igraph_vector_init(&modularity, 0);
      igraph_vector_init(&membership, 0);
      igraph_matrix_init(&memberships, 0, 0);
      high_resolution_clock::time_point t1 = high_resolution_clock::now();
      igraph_community_multilevel(&g, 0, &membership, &memberships, &modularity);
      high_resolution_clock::time_point t2 = high_resolution_clock::now();
      duration<double, milli> time_span = duration_cast<duration<double> >(t2 - t1);
      cout <<  "elapse time: " << time_span.count() << " miliseconds"<< endl;
      igraph_destroy(&g);
      igraph_vector_destroy(&modularity);
      igraph_vector_destroy(&membership);
      igraph_matrix_destroy(&memberships);

      return 0;
  }
  ```
- Original version
  ```console
  elapse time: 4887.1 miliseconds
  ```
- Our version
  ```console
  elapse time: 3401.65 miliseconds
  ```

## Prerequisite
- build-essential
- python-dev
- libxml2-dev
- zlib1g-dev
- libgmp3-dev

## Installation

The installation steps are the same as in the original igraph.

```console
$ chmod +x bootstrap
$ ./bootstrap
$ ./configure
$ make
$ make install
```

If you don't want to overwrite your original installed igraph, you can skip the `make install` and setup the dynamic library path toward to our `libigraph.so` (or `libigraph.a`) file. Since we keep each modified function stay in its original function prototype, you don't need to change your code to switch to our patched multilevel function.

Note that the path variable may variate in different OS:
- LINUX
  ```console
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/{YOUR WORKSPACE PATH}/igraph/src/.libs
  ```
- OSX
  ```console
  export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/{YOUR WORKSPACE PATH}/igraph/src/.libs
  ```

## Installation (Python)

It is recommended to install python wrapper under virtual environment.

```console
$ pip install https://github.com/igraph/python-igraph/archive/master.zip --global-option="--c-core-url=https://github.com/dacapo1142/igraph/archive/master.tar.gz"
```

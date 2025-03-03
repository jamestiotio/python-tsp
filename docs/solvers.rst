=======
Solvers
=======

This library has currently two classes of solvers: exact solvers and heuristics.

All solvers require at least a ``distance_matrix`` as input, which is an ``n x n`` numpy array containing the distance matrix for a problem with ``n`` nodes. This matrix can contain integers or floats and does not need to be symmetric. Other properties specific to each solver will be detailed below.

All of them also return a permutation of integers from ``0`` to ``n`` containing the best route found, plus a number indicating the route cost.

Exact solvers
=============

These are methods that always return the optimal solution of a problem. Their time requirements grow exponentially with the problem size, so use them in relatively small instances, wherein "small" depends on your resources. Problems with less than 15 nodes may be solvable in less than a few minutes.

Brute Force
-----------

This method checks all route permutations and returns the one with least cost. For a problem with ``n`` nodes, assuming the first one is fixed by the definition of the TSP, there are ``(n - 1)!`` total permutations, and this grows (even faster than) exponentially with ``n``.

.. code:: python

  from python_tsp.exact import solve_tsp_brute_force


  xopt, fopt = solve_tsp_brute_force(distance_matrix)


Dynamic Programming
-------------------

A Dynamic programming approach. It is faster than the Brute Force but may require more memory, which is controlled by the ``maxsize`` parameter.  When in doubt, leave it as default.

.. code:: python

  from python_tsp.exact import solve_tsp_dynamic_programming


  xopt, fopt = solve_tsp_dynamic_programming(distance_matrix, maxsize=None)

.. code:: rst
  
  Parameters
  ----------
  maxsize
    Parameter passed to ``lru_cache`` decorator. Used to define the maximum
    size for the recursion tree. Defaults to `None`, which essentially
    means "take as much space as needed".


Branch and Bound
----------------

A Branch and Bound approach, which may be more scalable than previous methods and not grow in time as fast as them. Courtesy of @luanleonardo.

.. code:: python

  from python_tsp.exact import solve_tsp_branch_and_bound


  xopt, fopt = solve_tsp_branch_and_bound(distance_matrix)


Heuristic solvers
=================

A rigorous definition of a heuristic can vary in different situations, but they can be viewed as algorithms that have no guarantees of finding the best solution, but usually return a good enough candidate in a more reasonable time for larger problems. So, if the exact solvers are unusable due to large processing time, these methods can be a good alternative.

All of the heuristics here use some form of neighborhood search or perturbation scheme. Both terms can be understood as synonyms: given a current permutation of nodes ``x``, we apply a perturbation -- effectively changing their order -- and obtain a new permutation ``x'``, which can be viewed as its "neighbor".

The perturbation schemes currently available are:

- ``two_opt``: the well-known `2-opt <https://en.wikipedia.org/wiki/2-opt>`_;
- ``ps1``, ``ps2``, ``ps3``, ``ps4``, ``ps5`` and ``ps6``: the PSX schemes work directly in the permutation space as shown in the figure below. Among these, the 2-opt is very close to the PS5 and it works very well in most instances, but sometimes other schemes may yield better results because their neighborhoods are different.

.. image:: ../figures/perturbation_schemes.png

Local Search
------------

Given a starting permutation, it creates new neighbors until no more neighbors are better than the current one, in which case we say it is a local optimum. 

Notice this local optimum may be different for distinct perturbation schemes and, of course, it may not be (most likely in large problems) the same as the global optimum.

.. code:: python

  from python_tsp.heuristics import solve_tsp_local_search

  xopt, fopt = solve_tsp_local_search(
      distance_matrix: np.ndarray,
      x0: Optional[List[int]] = None,
      perturbation_scheme: str = "two_opt",
      max_processing_time: Optional[float] = None,
      log_file: Optional[str] = None,
      verbose: bool = False,
  )

.. code:: rst

    Parameters
    ----------
    x0
        Initial permutation. If not provided, it starts with a random path

    perturbation_scheme {"ps1", "ps2", "ps3", "ps4", "ps5", "ps6", ["two_opt"]}
        Mechanism used to generate new solutions. Defaults to "two_opt"

    max_processing_time {None}
        Maximum processing time in seconds. If not provided, the method stops
        only when a local minimum is obtained

    log_file
        If not `None`, creates a log file with details about the whole
        execution

    verbose
        If true, prints algorithm status every iteration


Simulated Annealing
-------------------

An implementation of the `Simulated Annealing <https://en.wikipedia.org/wiki/Simulated_annealing>`_ metaheuristic. For users who do not care about its metaphor, it is enough to know that, being a metaheuristic, it may be slower, but it has better chances of avoiding getting trapped in local minima.


.. code:: python

  from python_tsp.heuristics import solve_tsp_simulated_annealing
  

  xopt, fopt = solve_tsp_simulated_annealing(
      distance_matrix: np.ndarray,
      x0: Optional[List[int]] = None,
      perturbation_scheme: str = "two_opt",
      alpha: float = 0.9,
      max_processing_time: Optional[float] = None,
      log_file: Optional[str] = None,
      verbose: bool = False,
  )

.. code:: rst

    Parameters
    ----------
    distance_matrix
        Distance matrix of shape (n x n) with the (i, j) entry indicating the
        distance from node i to j

    x0
        Initial permutation. If not provided, it starts with a random path

    perturbation_scheme {"ps1", "ps2", "ps3", "ps4", "ps5", "ps6", ["two_opt"]}
        Mechanism used to generate new solutions. Defaults to "two_opt"

    alpha
        Reduction factor (``alpha`` < 1) used to reduce the temperature. As a
        rule of thumb, 0.99 takes longer but may return better solutions, while
        0.9 is faster but may not be as good. A good approach is to use 0.9
        (as default) and if required run the returned solution with a local
        search.

    max_processing_time {None}
        Maximum processing time in seconds. If not provided, the method stops
        only when there were 3 temperature cycles with no improvement.

    log_file {None}
        If not `None`, creates a log file with details about the whole
        execution

    verbose {False}
        If true, prints algorithm status every iteration



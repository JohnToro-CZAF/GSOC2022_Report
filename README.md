# Final Report GSOC 2022
![[src/GSOC.jpeg]]
#### Organisation: [mlpack](https://github.com/mlpack)

#### Project: Enhance CMA-ES from existing implementation in ensmallen - a optimization header library.

#### Contributor: [John Hoang](https://github.com/JohnToro-CZAF)
#### Mentors: [Marcus Edel](https://github.com/zoq)

## Abstract
My proposal for Google Summer of Code 2022 [Enhance CMA-ES](http://www.2computing.net/article/Google-Summer-of-Code-Proposal/) was selected. CMA-ES abbreviated for Covariance Matrix Adaptation Evolution Strategy is a stochastically numerical optimization algorithm. Evolution Strategy algorithms find the candidate solutions in any type of optimization problem by constantly changing the original (randomly generated) population - an inspired biological evolution process. CMA-ES is one of the most robust algorithms for real-world problems. In recent years, several methods have been developed to increase the performance of the CMA-ES. There are variants out there of CMA-ES, but the most famous and is considered the original CMA-ES is (µ/µW, λ)-CMA-ES from Hansen and Ostermeier (1996). Adapting arbitrary normal mutation distributions in evolution strategies: The covariance matrix adaptation. A really nice tutorial about this paper: [link](https://arxiv.org/abs/1604.00772). The project is to aim for improve performance of the current [implementation](https://github.com/mlpack/ensmallen/tree/master/include/ensmallen_bits/cmaes) of (µ/µW, λ)-CMA-ES [ensmallen](https://github.com/mlpack/ensmallen)  and generalize the design of `cmaes` to add more efficient variants of CMA-ES easier. There are some popular variants are being used - since they are more effective in both accuracy and running time. **[IPOP-CMA-ES](https://ieeexplore.ieee.org/document/1554902)**,  **[saACM-ES](https://arxiv.org/abs/1204.2356)** and **[Active-CMA-ES](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.114.4239&rep=rep1&type=pdfn)** are three prominent variants that modify the original in different ways. 

## Goals
Since the CMA-ES has many default parameters that are calculated on the way of optimizing so this raise an issue of generalizing. An ideal base class `cmaes` should has these paremeters as default and could be splitted to separated parts - intialize, sample candidates, finally update paramters and results. Splitting to separated parts can benefit the generalization by adding new variants as different policies of the particular function. So this is the ultimate goal that I and my mentor - Marcus Edel aim for. 

## Result
Since the design is most difficult part, throughout the period of GSoC we experimented different design to fit to our planned variants - IPOP and other variants that modified only the part of updating the parameters. After many experiments we came up with a design that fits perfectlly to variants that only change the updating part and intialize population parameters. A fully functional `CMAES` base class was implemeted and throughly tested. Also I added two new updating policies - the main part of variants: VD-CMA and Sep-CMA. These two new variants are very effective in running time - their running time are linear with the solving dimension. With the new policy of initializing mutation's weights - `NegativeWeight`, now the algorithm can run in more consistent speed. Here is the result of running multiple combination of policies together - the experiment was done by running algorithms on logistic regression task with gisette dataset - [details](https://archive.ics.uci.edu/ml/datasets/Gisette). 

![[src/benchmark.png]]<br><hr>
All of my work in wrapped in one pull request only. You can have a look at this at:
[Pull Request #349 : Implementation of negative weights, Sep-CMA, VD-CMA. Sep and VD CMA achieving linear time complexity](https://github.com/mlpack/ensmallen/pull/349).
In this PR, I implemented:
* Negative weights policy (a recent improvement of default CMA-ES, more details is at https://arxiv.org/pdf/1604.00772.pdf - page 31) in `weight_init_policies/negative_weight.hpp`.
* Vd-CMA update policy: a linear time update covariance matrix as in  VD-CMA: Linear Time/Space Comparison-based Natural Gradient Optimization. The covariance matrix is limited as $C = D (I + v*v^t) D$, where D is a diagonal, v is a vector in `update_policies/vd_update.hpp`.
* Sep-CMA update policy: a linear time update covariance matrix by just updating the covariance matrix's diagonal as in Raymond Ros et al. in "A Simple Modification in CMA-ES Achieving Linear Time and Space Complexitys" in `update_policies/sep_update.hpp`.

Most imporant I changed the implementation of CMA main class, the purpose of the class is to store the default parameters of the algorithm once the optimizer is created and allow us to add new variants easier since I split the main algorithm into many parts. Since many improvements of the CMA-ES algorithm use the same set of parameters as the original one and many formulas look the same, this new CMAES class will reduce code replication. The code are in `cmaes.hpp` and `cmaes_impl.hpp`.

There is another closed pull request: the reason I closed this pull request because of the changes in design I made is too drastically so I decided to make a new pull request. Also the design of `cmaes` was changed in order to fit other sutiable(in time) variants which are different from the beginning expectation.
[Pull Request #346 : Implementation of CMAparameters class and negative weights](https://github.com/mlpack/ensmallen/pull/346)

## Future Work
I am planning to implement the IPOP restart strategy and Cholesky. The Cholesky update is only required to add a new update policy - this is farily easy but the IPOP needs more changes in the base class design or make a new separated algorithm with the `cmaes` base class as a base optimizer as the implementation of `adam` optimizer and `sgd` in `ensmallen` library. Yet to succesfully implement the IPOP I still need more time to research and get advices from others in commmunity. 

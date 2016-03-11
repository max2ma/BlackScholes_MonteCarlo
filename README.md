Monte Carlo Methods in Black-Scholes model
============
### Table of contents
[toc]


## Overviews
### Black-Scholes Model
[Black-Scholes model][Black-Scholes Model] which was first published by Fischer Black and Myron Scholes in 1973 has been a famous and basic mathematical model describing the beheaviour of the investment instruments in financial market. In general, there is one risky asset whose price following [geometric Brownian motion][geometric Brownian motion] and one riskless asset with a fixed interest rate. 

The geometric Brownian beheaviour of the price is described by a stochastic differential equation shown in eaution:

$$dS=rSdt+\sigma SdW_t$$

where $S$ is the price of the risky asset which is usually called stock price, $r$ is the fixed interest rate of the riskless asset, $\sigma$ is the volatility of the stock and $W_t$ is a [Wiener process][Wiener process].

According to Itô Lemma, there is an analytical solution for this stochastical  differential equation as following:

$$ S_{t+\Delta t}=e^{(r-\frac{1}{2}\sigma^2)\Delta t+\sigma\epsilon\sqrt{\Delta t} } $$

where $\epsilon\sim N(0,1)$, standard normally distribution.

### Call/Put Option
There are kinds of styles of [options][option], such as European vanilla option or Asian option which is one of the [exotic option][exotic option]. 
[Call options][Call options] and [put options][put options] are defined reciprocally. Given the basic parameters for an options, expire date and strike price, the call/put payoff price could be estimated. 
For European vanilla option,

$$P_{Call}=max\{S-K,0\}\\P_{put}=max\{K-S,0\}$$

where $S$ is the stock price at expire date and $K$ is strike price.
For Asian option,

$$P_{Call}=max\{\frac{1}{T}\int_0^TSdt-K,0\}\\P_{put}=max\{K-\frac{1}{T}\int_0^TSdt,0\}$$

where $T$ is the time period, $S$ is the stock price and$K$ is strike price.

[option]: https://en.wikipedia.org/wiki/Option_style
[exotic option]: https://en.wikipedia.org/wiki/Exotic_option

### Monte Carlo Method
[Monte Carlo Method][Monte Carlo] is one of the prominent approch to simulate stochastic process such as the stock price in Black-Scholes model, especially for exotic options which is usually not solvable analytically. In this project, Monte Carlo Method is implemented to estimate the payoff price of a given instrument in Black Scholes model. 

Given a time period, it has to be partitioned into M steps according to the style of option. M=1 for European option since the payoff price is independent of its prices before the expired date. At each time point, the stock price is determined by the stock price at previous time point and a normal distributed random number. The expectation of the payoff price is estimated by N parallel independent simulations.

The convergence of the result produced by Monte Carlo method is related to the computational cost $C=MN$. To reach a better convergence, $C$ is usually a very large number, which may reach $10^9$. 

### Normally Distributed Random Number Generation
The core feature of Monte Carlo method is random numbers. The quality of simulation results depends on the quality of the random number produced. The normal distributed random numbers in this project are generated by Mersenne-Twist algorithm and Box-Muller transformation. 

#### Mersenne-Twist
The [Mersenne Twister][Mersenne Twister] is an algorithm to generate uniformly distributed pseudorandom numbers. Its very long periodicity $2^{19937}-1$ makes it a suitable algorithm since there requests millions of random numbers in Monte Carlo method.

#### Box-Muller transform
[Box Muller transformation][Box Muller transformation] transforms a pair of independent, uniformly distributed random numbers in the interval (0,1) into a pairs of independent, standard, normally distributed random numbers, which are required for simulating Black Scholes model.
Given independent $U_1$,$U_2 \sim U(0,1)$, 

$$Z_1=\sqrt{-2ln(U_1)}cos(2\pi U_2)
\\ Z_2=\sqrt{-2ln(U_1)}sin(2\pi U_2)$$

then $Z_1$,$Z_2\sim N(0,1)$ independently.


## Getting Started Guide
The repository is called "blackScholes" in which there are two directories, "blackEuro" and "blackAsian", implementing European option and Asian option respectively. 
### Files Tree
```
blackScholes
│   README.md
│
└── headers
│   │   defTypes.h
│   │   RNG.h
│   │   RNG.cpp
│   │   stockData.h
│   │   stockData.cpp
│   └─  ML_cl.h
│
└── blackEuro
│   │   solution.tcl
│   │   blackEuro.h
│   │   blackEuro.cpp
│   │   main.cpp
│   │   blackScholes.h
│   └─  blackScholes.cpp
│
└── blackAsian
    │   solution.tcl
    │   blackAsian.h
    │   blackAsian.cpp
    │   testBench.h
    │   main.cpp
    │   blackScholes.h
    └─  blackScholes.cpp
```
File/Dir name  |Information
-------------- | ---
blackEuro(.cpp)| Top function for European options(kernel)
blackAsian(.cpp)|Top function for  Asian options(kernel)
solution.tcl   | Script to run sdaccel
blackScholes.cpp | BS model simulation, object created in top funtion
stockData.cpp	 | Basic stock datasets, object created in top function
RNG.cpp   | Random Number Generator class, objects created in object of blackscholes
main.cpp | Host code
testBench.h | Input parameters for kernel
ML_cl.h | CL/cl.hpp*

* "ML_cl.h" is the OpenCL library header file <CL/cl.hpp> of version 1.2.6 instead of the version 1.1 installed by sdaccel due to the pervious version causing some issues during compelling. See figure ![alt text][clerror]

[clerror]: https://github.com/KitAway/BlackScholes_MonteCarlo/blob/Readme/figures/header_failure.PNG

### Parameters
The values of the parameters for a given stock and option are list in ***"testBench.h"***. 

Parameters |  information
:-------- | :---
T	       |  time period
rate       |  interest rate of riskless asset
volatility |  volatility of the stock
S0		   |  initial price of the stock
K          |  strike price for the option
kernel_name | string stores the kernel name
The number of simulation $N$, number of time partition $M$ and all the other parameters related to the simulation are list in ***"blackScholes.cpp"***

Parameters |  information
:-------- | :---
MAX_NUM_RNG | (1) number of RNGs running parallelly
MAX_SAMPLE  | (2) number of simulations of each group
TIME_PAR    |  $M$
where $N=512*(1)*(2) $ and each parallel group assigned with a RNG runs 512 simulations.

### Run example
In each sub-directory, there is a script file called "solution.tcl". Open a terminal, run the command below and then the result of call/put payoff price estimation will be printed on standard IO.
> sdaccel solution.tcl

Due to some bugs in SDAccel, the kernel wirtten in C++ can't be emulated by CPU (See figure below) so that only hardware emulation is available. However, the hardware emulation usually takes long time. In order to have the results as far as possible, the computation cost $C$ should be as smaller as possible.

![alt text](https://github.com/KitAway/BlackScholes_MonteCarlo/blob/Readme/figures/CPU_emulation.PNG)

### Sample Output
For European option,

Input parameters |  value
:-------- | :---
T| 1
S0  | 100
K 	| 110
rate    |  5%
volatility | 20%
MAX_NUM_RNG | 8
MAX_SAMPLE  | 4
TIME_PAR | 1

Output |  value
:-------- | :--- 
call price| 6.048
put price | 10.65

For Asian option,

Input parameters |  value
:-------- | :---
T| 10
S0  | 100
K 	| 105
rate    |  1%
volatility | 15%
MAX_NUM_RNG | 2
MAX_SAMPLE  | 2
TIME_PAR	| 128

Output |  value
:-------- | :--- 
call price| 24.86
put price | 0.33

## Performance Metrics

As introduced before, computational cost $C=MN$ is an important factor that affects the both performance of the simulation and quality of the result. The time complexity of the algorithm is $O(MN)$ so that the performance for an algorithm is estimated by the simulation time per step, which is defined as:

$$t=T_s/C$$

For the algorithms in this repository, $t\approx1.25ns$ for 8 RNGs implementation. 


[Black-Scholes Model]: https://en.wikipedia.org/wiki/Black%E2%80%93Scholes_model 
[geometric Brownian motion]: https://en.wikipedia.org/wiki/Geometric_Brownian_motion	
[Wiener process]: https://en.wikipedia.org/wiki/Wiener_process
[Call options]: https://en.wikipedia.org/wiki/Call_option
[put options]: https://en.wikipedia.org/wiki/Put_option
[Mersenne Twister]: https://en.wikipedia.org/wiki/Mersenne_Twister
[Monte Carlo]: https://en.wikipedia.org/wiki/Monte_Carlo_method  
[Box Muller transformation]: https://en.wikipedia.org/wiki/Box%E2%80%93Muller_transform


---
layout: post
title: Hidden Markov models - Part I 
excerpt_separator:  <!--more-->
---

Hidden Markov models refers to a class of probabilistic graphical models that describes the probabilistic dependence between the latent state variable \\(x_{t}\\) and the observed measurement \\(y_t\\) for \\(t=1,\ldots,T\\).

Hidden Markov models are charachterized by two conditional independence assumption. First of all the latent state is Markovian process, which means "the conditional probability distribution of future states of the process (conditional on both past and present values) depends only upon the present state; that is, given the present, the future does not depend on the past." This means that knowing the current state is enough to describe the conditional distribution of futures state given current and past states and the knowledge of past states would have no additional information about future states. More formally
\\[p(x_{t+1}|x_{t}, \ldots, x_{1}) = p(x_{t+1}|x_{t})\\]
Second, observation are conditionally independent, which means that the conditional disitribution of the current observation (conditional on all states and observations) only depends on the present state. This means that knowing the current state we can describe the conditional distribution of the observation and conditioning on more observations or states would not change this distribution. We can write this formally as
\\[p(y_t|x_T,\ldots,x_1,y_T,\ldots,y_1)=p(y_t|x_t)\\]
These assumptions imply a factoriation of the joint density of observations and states
\\[p(y_T,\ldots,y_1,x_T,\ldots,x_1)=p(y_1|x_1)p(x_1) \prod \limits_{t=2}^{T} p(y_t|x_t)p(x_t|x_{t-1}),\\]
which means that we can define the joint disitriution of the obserations and states (hence a hidden Markov model) by spefifying the observation density \\(p(y_t|x_t)\\), the transition density \\(p(x_t|x_{t-1})\\) and the initial distribution of the state \\(p(x_1)\\). 

General hidden Markov models can be specified
\\[y_t \sim p_{\theta}( \cdot | x_t )  \quad \text{observation density} \\]
\\[x_{t+1} \sim p_{\theta}(\cdot | x_{t})\quad \text{transition density} \\]
\\[x_1 \sim p_{\theta}(\cdot),\\]
where \\(x_t\\) is the scalar or vector of hidden states at time \\(t\\), while \\(y_{t}\\) is the scalar or vector observation at time \\(t\\).  We can think about \\(y_t\\) as a noisy measurement of the latent state \\(x_t\\). The dynamics of the state is governed by the transition density \\(p_{\theta}(x_{t+1}|x_t)\\), while the dependence between \\(y_t\\) and \\(x_t\\) or alternatively the distirbution of noise is determined by the observation density \\(p_{\theta}(y_{t}|x_t)\\). Note that both of these densities can depend on hyper paramters \\(\theta\\).

![hmm](https://github.com/istvanbarra/istvanbarra.github.io/blob/master/_posts/hmm.png)

In hidden Markov models we are usually interested in three quantities. First of all, we are interested in the dsitribution of the current state conditional on the current nad past observations
\\[p(x_t | y_{t},\ldots, y_{1}),\\]
which is called the filtering density. Another interesting quantity is the disitribution of the current state using all available data (past, current, future)
\\[p(x_t | y_{T},\ldots, y_{1}),\\]
which is called the smoothing density and we can think about it as the revision of the filtering density after observing new data points. Finally we are interested in parameter estimation which means we are interested in $\theta$ the hyper paramteres of the model.

# Discrete state

The simplest HMM model example is one where both observations and states are discrete variables, hence the observation and
transition distributions are discrete distributions. 

Consider Carol a Candy Crush player who lives in a country called BinaryWeatherLand. In BinaryWeatherLand there are only sunny (S) and rainy (R) days. We also know from Wikipedia that rainy days and sunny days are persistent, so after a rainy day it is more likely to have a rainy day and a sunny day is ususally followed by a sunny day. Weather forecasters from BinaryWeatherLand estimated the following transition probabilities from thousands of years of observations. 
\\[T= \begin{bmatrix} t_{SS} & t_{SR} \\\ t_{RS} & t_{RR} \end{bmatrix} = \begin{bmatrix} 0.9 & 0.1 \\\ 0.2 & 0.8 \end{bmatrix},\\]
where 
\\[ t_{ij}=p(x_{t+1}=j|x_{t}=i). \\]
Note that $T$ completly descirbes the transition density as it covers all the possible tranistion combinations. 
Carol is interested in two activities playing Candy Crush and meeting with her friends, everyday she wakes up and based on the weather she decides if she plays Candy (C), pays (P) in the game or meets with her friends (F). She has a habit that on rainy days she playes more Candy Crush and it is also more likely that she will spend in the game, while on sunny days she goes out with her friends. This pattern described by 
\\[ Z= \begin{bmatrix} z_{SP} & z_{SC} & z_{SF} \\\ z_{RP} & z_{RC} & z_{RF} \end{bmatrix}=\begin{bmatrix}0.2 & 0.3 & 0.5  \\\ 0.6 & 0.3 & 0.1 \end{bmatrix},\\]
where 
\\[ z_{ij}=p(y_{t}=j|x_{t}=i). \\]
We can look up Carol in the f_activity_summary table so every day we can check if she has played the game, has spent or 
didn't play at all and met with her friends. 

## Filtering


The first question we are interested in is if today is a rainy day in BinaryWeatherLand condition on Carol's activity.  Before seeing any data we believe that a rainy or a sunny day is equally likely. We run a Hive query and we find out that we have only one data point about Carol and it seems she didn't play today \\(y_1=F\\). We know that \\(p(y_1|x_{1}^{S})=0.5\\) and \\(p(y_1|x_{1}^{R})=0.1\\). A naive answer is that today is probably a sunny day in BinaryWeatherLand as the likelihood of a sunny day is more likely than a rainy day given  our observation \\(p(y_1|x_{1}^{S})>p(y_1|x_{1}^{R})\\). Luckily in this simple case the naive answer is the correct answer as more formally we have
\\[\begin{align} p(x_{1}^{S}|y_1)&=\frac{p(x_{1}^{S},y_1)}{p(y_1)}=\frac{p(y_1|x_{1}^{S})p(x_{1}^{S})}{p(y_1|x_{1}^{S})p(x_{1}^{S})+p(y_1|x_{1}^{R})p(x_{1}^{R})}  \\\ &=\frac{0.5\times0.5}{0.5\times 0.5 +0.1\times0.5}=0.833 \end{align}\\]
and
\\[ p(x_{1}^{R}|y_1)=\frac{p(x_{1}^{R},y_1)}{p(y_1)}=\frac{p(y_1|x_{1}^{R})p(x_{1}^{R})}{p(y_1|x_{1}^{S})p(x_{1}^{S})+p(y_1|x_{1}^{R})p(x_{1}^{R})}=\frac{0.1\times0.5}{0.5\times 0.5 +0.1\times0.5}=0.166\\]
Next day we run the hive query again and we find that Carol has spent some money on gold bars (\\(y_2=P\\)). We know that \\(p(y_2|x^{S}_2)=0.2\\) and \\(p(y_2|x^{R}_2)=0.6\\) so our naive approach would suggest a rainy day is more likely, however we also know that after sunny day it is more probable to have a sunny day \\(p(x^{S}_2|x^{S}_1)=0.9\\) \\(p(x^{R}_2|x^{S}_1)=0.1\\). The question how we can combine these probabilities in a consistent way. We use basic probability theory to achive this
\\[ p(x^{S}_2|y_1,y_2)=\frac{p(x^{S}_2,y_1 ,y_2)}{p(y_1,y_2)}=\frac{p(y_2|x^{S}_2,y_1)p(x^{S}_2|y_1)p(y_1)}{p(y_2|y_1)p(y_1)}=\frac{p(y_2|x^{S}_2)p(x^{S}_2|y_1)}{p(y_2|y_1)}= \frac{p(y_2|x^{S}_2)\sum \limits_{i \in {S,R}}p(x^{S}_2,x^{i}_1 |y_1)}{p(y_2|y_1)}=\frac{p(y_2|x^{S}_2)\sum \limits_{i \in {S,R}}p(x^{S}_2| x^{i}_1) p( x^{i}_1|y_1)}{p(y_2|y_1)}\\]
and 
\\[ p(x^{R}_2|y_1,y_2)=\frac{p(x^{R}_2,y_1 ,y_2)}{p(y_1,y_2)}=\frac{p(y_2|x^{R}_2,y_1)p(x^{R}_2|y_1)p(y_1)}{p(y_2|y_1)p(y_1)}=\frac{p(y_2|x^{R}_2)p(x^{R}_2|y_1)}{p(y_2|y_1)}= \frac{p(y_2|x^{R}_2)\sum \limits_{i \in {S,R}}p(x^{R}_2,x^{i}_1 |y_1)}{p(y_2|y_1)}=\frac{p(y_2|x^{R}_2)\sum \limits_{i \in {S,R}}p(x^{R}_2| x^{i}_1) p( x^{i}_1|y_1)}{p(y_2|y_1)}\\]
where 
\\[p(y_2|y_1)=\sum \limits_{j \in {S,R}} \sum \limits_{i \in {S,R}} p(y_2,x^{i}_2,x^{j}_1|y_1) =\sum \limits_{j \in {S,R}}  p(y_2|x^{j}_2)\sum \limits_{i \in {S,R}}p(x^{j}_2| x^{i}_1) p( x^{i}_1|y_1).\\]
Note that we have expressed all these probabilities as the function of the previous filtering density, the observation density and the transition density. So know we can substitute in these values in the formula and get the following answer
\\[ p(y_2|y_1)=0.2\times(0.9\times0.833 + 0.2\times0.166 ) + 0.6\times(0.1\times0.833  + 0.8\times0.166  )=0.2866\\]
\\[ p(x^{S}_2|y_1,y_2)=\frac{0.2\times(0.7\times0.833 + 0.2\times0.166 )}{0.2866} =0.5465\\]
\\[ p(x^{R}_2|y_1,y_2)=\frac{0.6\times(0.3\times0.833  + 0.8\times0.166 )}{0.2866} =0.4534\\]
In general we can get the filtering density
\\[ p(x_t | y_{1:t}), \\]
using the following recursion (where \\(a_{1:t}=(a_1, \ldots,a_t)\\))
\\[ p(x^{i}_t | y_{1:t})=\frac{p(x^{i}_t, y_{1:t})}{p(y_{1:t})}=\frac{p(y_t| x^{i}_t,y_{1:t-1})p(x^{i}_t|y_{1:t-1})p(y_{1:t-1})}{p(y_{t}|y_{1:t-1})p(y_{1:t-1})}=\frac{p(y_t| x^{i}_t)p(x^{i}_t|y_{1:t-1})}{p(y_{t}|y_{1:t-1})}=\frac{p(y_t| x^{i}_t)\sum \limits_{j} p(x^{i}_{t},x^{j}_{t-1}|y_{1:t-1})}{p(y_{t}|y_{1:t-1})} =\frac{p(y_t| x^{i}_t)\sum \limits_{j} p(x^{i}_{t}|x^{j}_{t-1})p(x^{j}_{t-1}|y_{1:t-1})}{p(y_{t}|y_{1:t-1})},\\]
where
\\[ p(y_{t}|y_{1:t-1}) =\sum \limits_{i} p(y_t| x^{i}_t)\sum \limits_{j} p(x^{i}_{t}|x^{j}_{t-1})p(x^{j}_{t-1}|y_{1:t-1}) \\]

## Smoothing

Now let's consider the smoothing problem
\\[ p(x_t | y_{1:T}). \\]
On the last day the the smoothing and filtering density ofthe state is the same, so lets check 
\\[ p(x^{S}_{1}|y_1,y_2) = \frac{p(x^{S}_1,y_1,y_2)}{p(y_1,y_2)} =\frac{p(y_2|x^{S}_1)p(x^{S}_1|y_1)p(y_1)}{p(y_2|y_1)p(y_1)}=
   \frac{\sum \limits_{i} p(y_2, x^{i}_2 |x^{S}_1)p(x^{S}_1|y_1)}{p(y_2|y_1)}= \frac{\sum \limits_{i} p(y_2| x^{i}_2)p(x^{i}_2 |x^{S}_1)p(x^{S}_1|y_1)}{p(y_2|y_1)}
\\]
\\[ p(x^{R}_1|y_1,y_2) =\frac{p(x^{R}_1,y_1,y_2)}{p(y_1,y_2)} =\frac{p(y_2|x^{R}_1)p(x^{R}_1|y_1)p(y_1)}{p(y_2|y_1)p(y_1)}=
   \frac{\sum \limits_{i} p(y_2, x^{i}_2 |x^{R}_1)p(x^{R}_1|y_1)}{p(y_2|y_1)}= \frac{\sum \limits_{i} p(y_2| x^{i}_2)p(x^{i}_2 |x^{R}_1)p(x^{R}_1|y_1)}{p(y_2|y_1)}\\]
Pluging in the numbers yield to
\\[ p(x^{S}_1|y_1,y_2) =\frac{(0.2\times0.9+0.6\times0.1)\times 0.833 }{0.2866} =0.6976 \\]
\\[ p(x^{R}_1|y_1,y_2)=\frac{(0.2\times0.2+0.6\times0.8)\times 0.166 }{0.2866} = 0.3023 \\]
In general we can solve the smoothing problem, using the following backward recursion
\\[ p(x^{i}_t | y_{1:T})=\frac{p(x^{i}_t, y_{1:T})}{p(y_{1:t})}= \frac{p( y_{t+1:T} |x^{i}_{t},y_{1:t})p(x^{i}_t|y_{1:t})p(y_{1:t})}{p(y_{t+1:T}|y_{1:t})p(y_{1:t})}=\frac{p( y_{t+1:T} |x^{i}_{t})p(x^{i}_t|y_{1:t})}{p(y_{t+1:T}|y_{1:t})} \\]
\\[p( y_{t:T} |x^{i}_{t-1})=\sum \limits_{j} p(y_{t+1:T},y_t,x^{j}_t|x^{i}_{t-1})=\sum \limits_{j} p(y_{t+1:T}|y_t,x^{j}_t,x^{i}_{t-1})p(y_t|x^{j}_t,x^{i}_{t-1})p(x^{j}_t|x^{i}_{t-1})=\sum \limits_{j} p(y_{t+1:T}|x^{j}_t)p(y_t|x^{j}_t)p(x^{j}_t|x^{i}_{t-1})\\]
The above forward and backward recursion is called the forward backward algorithm.

## Estimation

Using the forward pass we can evaluate the likelihood \\(p(y_{1:t})\\)which makes both frequentist 
\\[ \hat{\theta} = \arg \max_{\theta} p_{\theta}(y_{1:T})\\] and Bayesian 
\\[ p(\theta|y_{1:T}) \propto p_{\theta}(y_{1:T}) p(\theta)\\]
parameter estimation feasible. 



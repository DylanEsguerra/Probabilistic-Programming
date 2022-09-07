# Probabilistic Programming for Sars-Cov-2 Antibody regressions 


## Motivation

From existing SARS-CoV-1 antibodies, LLNL researchers have introduced different combinations of multiple amino acid mutations that should increase their binding affinity to the SARS-CoV-2 virus. For each potential SARS-CoV-2 antibody, manufacturability data was collected, and this was used to model the effects of the single point mutations on the manufacturability/binding metrics.

We are interested in evaluating how using different tools and samplers will effect our coefficients and the run time of each model. 

## Probabilistic Programming Languages

Probabilistic programming is a tool for statistical modeling where a model is defined in terms of random variables and inference is run automatically. 

We will be evaluating two models in five different PPL's available with python. For most PPL's the default sampler for MCMC is NUTS. We aimed to fit all the models in NUTS and used other samplers when appropriate. 

\begin{center}
\begin{tabular}{||c c c c c||} 
 \hline
 Numpyro & Pyro & PyMC3 & Edward2 & Tensorflow Probability \\ [0.5ex] 
 \hline 
 NUTS & NUTS & NUTS and MH & NUTS and HMC & NUTS and HMC \\ 
 \hline
\end{tabular}
\end{center}

## The Data
This dataset consists of 97 antibody observations and 92 amino acid mutations used as features. We want to model the effect of these mutations on binding affinity to the SARS-CoV-2 virus. 

##The Models
We chose to fit two Bayesian models, the first being a simple regression model and the second being a hierarchical model with both random slopes and intercepts.  


### Simple Bayesian Model
The first model gives normal priors on the slopes and intercept and an Inverse gamma prior on the models variance. These were chosen do to the analytical tractability given by conjugacy should comparisons want to be made between the inference preformed by the PPL's and a hand coded Gibb's sampler. Hyper-parameters were chosen to reflect the data particularly for the intercept. 

     $y_{i}|\beta,\alpha,\sigma \sim N({x_{i}}^{T} \beta + \alpha, \sigma^2), i = 1,......,n$

     $\beta \sim N(0,1)$

     $\alpha \sim N(70,10)$

     $\sigma^2 \sim IG(1,1)$

### Hierarchical Bayesian Model

This model was chosen to explore the effect of antibody clusters on our coefficients. Our data set can be divided into two distinct antibody clusters based on similarities between antibodies. We want to see how mutations behave on different antibody clusters which can be thought of as backbones to witch the mutations are applied.  

Hierarchical models also known as partially pooled models are good for modeling data with different groups because they allow for between group variation but still link the groups by a common prior distribution. 


     $y_{ij}|\beta,\alpha,\sigma \sim N({x_{ij}}^{T} \beta_{i} + \alpha_{i}, \sigma^2), i = 1,2, j = 1,....n_{i}$

     $\beta_{i} \sim N(\mu_{b},\tau_{b})$

     $\alpha_{i} \sim N(\mu_{a},\tau_{a})$

     $\sigma^2 \sim IG(100,100)$

     $\mu_{b} \sim N(0,10^4)$

     $\tau_{b} \sim IG(100,100)$

     $\mu_{a} \sim N(0,10^4)$

     $\tau_{a} \sim IG(100,100)$

## Results

### Coefficients

### Run time

Each model was run for 5,000 samples and 2,500 burn in steps. This was chosen as most models appeared to converge well at this length and consistency of iterations is important for run time comparisons. However it must be noted that not all samplers are the same in the amount of iterations needed to reach convergence. Note that the NUTS sampler seemed to have already converged for both models in as little as 1,000 steps where HMC and especially the Metropolis Hastings algorithm need more. 

\begin{center}
\begin{tabular}{ |p{2cm}||p{2cm}|p{2cm}|p{2cm}|p{2cm}|p{2cm}| }
 \hline
 \multicolumn{6}{|c|}{Simple Model} \\
 \hline
  & Numpyro & Pyro & pyMC3 & Edward2 & TFP \\
 \hline
 Script   & NUTS: 17.06 &NUTS: 1390.59& NUTS: 87.74 MH: 6.46 & NUTS:178.40 HMC:15.48  &  NUTS:176.11 HMC:9.70 \\
 \hline
 Notebook & NUTS: 17.00 &NUTS: 436.46& NUTS: 33.51 MH: 9.01 & NUTS:170.71 HMC:15.12  &  NUTS:171.58 HMC:9.57 \\
 \hline
\end{tabular}
\end{center}


\begin{center}
    \begin{tabular}{ |p{2cm}||p{2cm}|p{2cm}|p{2cm}|p{2cm}|p{2cm}| }
    \hline
    \multicolumn{6}{|c|}{Hierarchical Model} \\
    \hline
    & Numpyro & Pyro & pyMC3 & Edward2 & TFP \\
    \hline
    Script   & NUTS: 54.53& NUTS: 1815.60 & NUTS: N/A MH: 50.42 & NUTS: 2053.96 HMC: 31.56  &  NUTS: 2086.73 HMC: 15.48 \\
    \hline
    Notebook & NUTS: 56.54 & NUTS: 2236.87 & NUTS: N/A MH:29.26 & NUTS: 1968.84 HMC: 30.06  &  NUTS: 1981.48 HMC: 21.88 \\
    \hline
    \end{tabular}
\end{center}

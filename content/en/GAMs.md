+++
title = 'GAMs'
date = 2024-12-18
categories = ["Statistical Modelling"]
tags = ["GAMs", "GLMs"]
keywords = ["GAMs"]
description = "SEO Description Here"
draft = false
[params.math]
  math = true
+++


![Image](/image/anh-vy--kJ542I1oVU-unsplash.jpg)
[Image by Anh Vy.](https://unsplash.com/@anhvygor)


# GAMs

In this article we will cover Generalised Additive Models (GAMs). We'll cover the base case by modelling a response variable with a univariate smooth function. We'll then build on this by incorporating multiple exogenous variables to create additive models. After then, the GAM can be covered.

For more details about this methods please read the book by Simon N. Wood about Generalised Additive Models.


**GAMS:** GAMs are a form of generalised linear models with linear response variables that depend on unknown smooth functions of some exogenous variables. These forms of models were initially developed to blend the properties of Generalised Linear Models (GLMs) with additive models.

The models goes as follows, given a univariate response variable Y and predictor (exogenous) variables $x_i, i \in \{1, ..., m\}$. An exponential family distribution is chosen for $Y$ along with a link function $g$ which relates the expected value of Y to the predictor variables via 
$$g(\mathbb{E}[Y]) = \beta_0 + f_1(x_1) + ... + f_m(x_m)$$
where $f_i$ are functions with a specified parametric forms (for example, $f_i$  may follow a polynomial or an unpenalised regression spline of a variable) or may be specified non-parametrically or semi-parametrically as simply smooth functions to be estimated in a non-parametric way. Due to the relaxed assumptions of the relationship between the response and the predictors, it provides a better fit to the data than the parametric models but with a loss of interpretability.

The idea behind GAMs comes from the **Kolmogorov-Arnold representation theorem** which states that any multivariate continuous function can be represented as sums and compositions of univariate functions:
$$f(x) = \sum_{q=0}^{2n} \Phi_q\left(\sum_{p=1}^n \phi_{q, p}(x_p)\right)$$
This theorem states a representation of this form exists but it provides no method to construct this function in practice. Proofs to find this representation are complex and often require the use of functions with a fractal structure. While fractals are mathematically interesting, they are not practical for models due to their complexity and computational complexity. Consequently, Kolmogorov-Arnold model based methods are not suitable for practical modelling. In modelling, you tend to focus on functions which are simple and easily interpretable. Fractal based methods are the opposite to this.

As a result, GAMs drop the outer sum focus on creating functions of a simpler class:
$$f(x) = \Phi \left(\sum_{p=1}^n \phi_p (x_p)\right)$$
where $\Phi$ is a smooth monotonic function. In practice, the link function $g$ is often the inverse of this function and is written as $$g(f(x)) = \sum_i f_i (x_i).$$When this model is tasked to find the expectation of a response variable $Y$, it can be written in the form:
$$g(\mathbb{E}[Y]) = \beta_0 + f_1(x_1) + ... + f_m(x_m).$$

## Univariate Smooth Functions

Given we are trying to represent a response distribution using a univariate smooth function:
$$\begin{equation}y_i = f(x_i) + \epsilon_i\end{equation}$$
where $\epsilon_i$ is a random variable that follows a normal distribution.

To estimate $f$, we can try to represent it in a way such that it becomes a linear model. To do this, we choose a **basis** which defines the space of functions for $f$. To create this basis we will need to choose some **basis functions**. If $b_i$ is the ith basis function, then $f$ can be represented as:
$$f(x) = \sum_{i=1}^q b_i(x) \beta_i$$
for some scalar parameters $\beta_i$. Substituting this equation into the previous one gives a linear function.

For example, if we believed $f$ could be represented as a 3rd order polynomial function. Then a basis for this space can be constructed using $b_1(x) = 1, b_2(x) = x, b_3(x) = x^2, b_4(x) = x^3$ to give us a function of the form:
$$f = \beta_1 + \beta_2x + \beta_3x^2 + \beta_4 x^3.$$
Polynomial bases are useful in situations which focuses on the properties of $f$ in the vicinity of a single point but when the interest relates to $f$ over the whole domain, the polynomial bases underperform. As the relationships become more complicated, the polynomials tend to oscillate and experience Runge's Phenomenon at the boundaries. In these situations, splines are recommend to be used due to theoretic properties.

A univariate function can be expressed as a **cubic spline**. A cubic spline is a curve made up of sections of cubic polynomials joined together such that they are continuous in value in the first and second derivatives. The points which join the curves together are called the **knots** of the spline.

In a conventional spline, the knots are distributed at any point where a datum exists. For regression splines, the knots are evenly spaced through a range of observed x values or placed at quantiles of the distribution of the unique x values.

 If a cubic spline is used as a basis for the function , then the function for the response variable  can be written as a linear model  where the ith row is expressed as 
$$X_i = [1, x_i, R(x_i, x_i^*), R(x_i, x_2^*), ..., R(x_i, x^*_{q-2})].$$As a result, the parameters of this model can be estimated using the least squares:
$$\min_\boldsymbol{\beta} \|\textbf{y} - X \boldsymbol{\beta}\|^{2},$$
to give us 
$$\boldsymbol{\beta} = (X^T X)^{-1} X^T Y.$$
The next step is to choose a suitable basis dimension $q$. One way to go about choosing $q$ is with backwards selection, however, this method will not guarantee the dots will be evenly spaced at the end of the process. An alternative method suggests keeping the base dimension fixed and adding a "wigglinss" penalty to the least squares fitting objective giving us this minimisation problem:
$$\min_\boldsymbol{\beta} \|\textbf{y} - X \boldsymbol{\beta}\|^{2} + \lambda \int_0^1[f''(x)]^2dx,$$
The tradeoff between the model fit and model smoothness is controlled by the smoothing parameter $\lambda$. As $\lambda \rightarrow \infty$, the model leads to a straight line estimate and at $\lambda =0$ the regression spline will be not be penalised.

Since $f$ is linear in the parameters $\beta_i$ , the penalty can always be written as a quadratic form in $\boldsymbol{\beta}$.
$$\int_0^1[f''(x)]^2 dx = \boldsymbol{\beta}^T S \boldsymbol{\beta},$$
where $S$ is a matrix of known coefficients. Rewriting our new least squares objective with this new term gives:
$$\min_\boldsymbol{\beta} \|\textbf{y} - X \boldsymbol{\beta}\|^{2} + \lambda \boldsymbol{\beta}^T S \boldsymbol{\beta},$$
Finding an estimator for the parameters gives:
$$\boldsymbol{\hat{\beta}} = (X^TX + \lambda S)^{-1} X^T \textbf{y}.$$
Now that we have formed an objective function to minimise and a formula for our estimator $\boldsymbol{\hat{\beta}}$ , we need to find an effective way to find a suitable value for the parameter $\lambda$. In an ideal situation, we will choose $\lambda$  such that $\hat{f}$ (our function) is as close as possible to the true function $f$. A suitable criterion we might then choose to minimise takes the form
$$M = \frac{1}{n} \sum_{i=1}^n (\hat{f}_i - f_i)^2,$$
where $f_i = f(x_i)$ and $\hat{f}_i = \hat{f}(x_i)$. However, $f$ is unknown so $M$ cannot be used but it is possible to find an estimate for it. Define the ordinary cross validation (OCV) score as:
$$\mathcal{V}_0 = \frac{1}{n} \sum_{i=1}^n (\hat{f}_i^{[-i]} - y_i)^2,$$
where $\hat{f}_i^{[-i]}$ is the model fitted to all data points excluding datum $x_i$. This score fits each model with all the data except the datum $x_i$, then finds the difference between the missed out datum's predicted and true value.
$$\begin{align}\mathcal{V}_0 &= \frac{1}{n} \sum_{i=1}^n (\hat{f}_i^{[-i]} - f_i - \epsilon_i)^2 \\ &= \frac{1}{n} \sum_{i=1}^n (\hat{f}_i^{[-i]} - f_i)^2 - (\hat{f}_i^{[-i]} - f_i)\epsilon_i + \epsilon_i^2\end{align}$$
Since $\mathbb{E}[\epsilon_i] = 0$ and $\epsilon_i$ and $\hat{f}_i^{[-i]}$ are independent, the second term vanishes when taking the expectation of the OCV score. This is only possible because we fitted our model without using datum $x_i$.
$$\mathbb{E}[\mathcal{V}_0] = \frac{1}{n} \mathbb{E} \left[ \sum_{i=1}^n(\hat{f}_i^{[-i]} - f_i)^2  \right] + \sigma^2$$
Since $\hat{f}_i^{[-i]} \approx \hat{f}$ and under a large sample achieves equality, we can say $\mathbb{E}[\mathcal{V}_0] = \mathbb{E}[M] + \sigma^2$ and achieves equality with a large sample. Consequently, we need to choose a lambda that minimises $\mathbb{E}[\mathcal{V}_0]$.

The issue with this method comes with estimating $\mathcal{V}_0$ by leaving out 1 datum at a time. Luckily, there is an alternative formula for the cross validation score:
$$\mathcal{V}_0 = \frac{1}{n} \sum_{i=1}^n (y_i - \hat{f}_i)^2/(1-A_{ii})^2.$$
where $\hat{f}$ is the estimate fitting all the data, $A$ is the influence matrix.

Alternatively, a generalised cross validation loss (GCV) can be constructed by
$$\mathcal{V}_g = \frac{n \sum_{i=1}^n (y_i - \hat{f}_i)^2}{[tr(I-A)]^2}.$$
The GCV has many computational advantages over the OCV and it retains the property of minimising the $\mathbb{E}[M]$.

To minimise the OCV or the GCV with respect to $\lambda$, numerical optimisation methods will need to be applied such as grid search and quasi-newton methods like BFGS.

## Additive Models

Now that we know how to solve the problem using a univariate smooth function, suppose the response variable $y$ has two predictor variables $x$ and $z$. This gives an additive model taking the form:
$$y_i = f_1(x_i) + f_2(z_i) + \epsilon_i,$$
where $f_j$ are smooth functions and $\epsilon_i$ are i.i.d $N(0, \sigma^2)$ random variables. 

Before continuing, it is important to mention this model suffers from identifiability problems: $f_1$ and $f_2$ are only estimable to within an additive constant. This means that a constant could be added from $f_1$ and subtracted from $f_2$ without changing the model predictions. As a result, identifiability conditions must be applied to the model before fitting.

Given that the identifiability issue has been addressed, the additive model can be represented using a regression spline estimating the penalised least squares and the degree of smoothing can be estimated by cross validation.

Each smooth function can be represented using the splines discussed above giving us the equation:
$$f_1(x) = \delta_1 + x\delta_2 + \sum_{j=1}^{q_1 - 2} R(x, x_j^*) \delta_{j+2}$$
and
$$f_1(x) = \gamma_1 + z\gamma_2 + \sum_{j=1}^{q_2 - 2} R(z, z_j^*) \gamma_{j+2}$$
where $\delta_j$ and $\gamma_j$ are unknown parameters, $x_j^*$ and $z_j^*$ are knot locations.

The identifiable problem with the additive model mean $\delta_1$ and $\gamma_1$ are confounded. The simplest way to solve this problem is by constraining one of the parameters to 0. Under this constraint, the model can be written as a linear model $\textbf{y} = X \boldsymbol{\beta} + \boldsymbol{\epsilon}$ where the ith row of the matrix $X$ is written as:
$$X_i = [1, x_i, R(x_i, x_1^*), R(x_i, x_2^*), ..., R(x_i, x_{q_1 - 2}^*), z_i, R(z_i, z_2^*), ..., R(z_i, z_{q-2}^*)]$$
with the parameter vector $\boldsymbol{\beta}$ written as $\boldsymbol{\beta} = [\delta_1, \delta_2, ..., \delta_{q_1}, \gamma_2, \gamma_{q_2} ]^T.$

As we also did in the last section, the wiggliness of a function can be measured using:
$$\int_0^1[f_1''(x)]^2 dx = \boldsymbol{\beta}^T S_1 \boldsymbol{\beta}, \quad \text{and} \quad \int_0^1[f_2''(x)]^2 dx = \boldsymbol{\beta}^T S_2 \boldsymbol{\beta}.$$
To fit the model, we need to minimise the penalised least squares objective.
$$\| \textbf{y} - X \boldsymbol{\beta}\|^2 + \lambda_1 \boldsymbol{\beta}^T S_1 \boldsymbol{\beta} + \lambda_2 \boldsymbol{\beta}^T S_2 \boldsymbol{\beta},$$
where $\lambda_1$ and $\lambda_2$ are the smoothing parameters.

## Generalised Additive Models
GAMs follow from the additive models and take a similar form to generalised linear models where the response variables are known to follow some distribution from the exponential family or have a known mean variance relationship.

Suppose we choose to model our response variables with the model:
$$log(\mathbb{E}[y_i]) = f_1(x_1) + f_2(x_2) \quad \text{where} \quad y_i \sim \text{Gamma}.$$
With this model, we cannot take the same approach as above. The additive model was fitted with the penalised least squares and the GAM will be fit by a penalised iterative least squares.  

The reason behind the change in our objective criterion is because our the residuals of our model is no longer assumed to be normally distributed. Consequently the least squares is no longer the optimal criterion because it does not properly account for the variance in the distribution. This means that an alternative criterion needs to be used and this will be the maximised penalised likelihood. Often this takes the form of the log-likelihood because we assume that the residuals of the model come from an exponential family such as a Gaussian, Binomial, Exponential, and Poisson distribution.

To fit the model with a log-likelihood objective, the penalised iteratively re-weighted least squares (P-IRLS) scheme is run till convergence.

1. Given at the kth iteration, the parameters $\boldsymbol{\beta}^{[k]}$ has been generated and we have the mean response vector $\boldsymbol{\mu}^{[k]}$, calculate:
$$w_i = \frac{1}{V(\mu_i^{[k]})g'(\mu_i^{[k]})} \text{ and } z_i = g(\mu_i^{[k]})(y_i - \mu_i^{[k]}) + X_i \boldsymbol{\beta}^{[k]}$$
where $\text{var}(Y_i) = V(\mu_i^{[k]}) \phi$, $V(\mu_i^{[k]})$ denotes the variance function, $\phi$ is the dispersion parameter,  and $X_i$ is the ith row of $X$. 
2. Minimise 
$$\| \sqrt{W} (z - X\boldsymbol{\beta})\|^2 + \lambda_1 \boldsymbol{\beta}^T S_1 \boldsymbol{\beta} + \lambda_2 \boldsymbol{\beta}^T S_2 \boldsymbol{\beta}$$
w.r.t $\boldsymbol{\beta}$ to obtain $\boldsymbol{\beta}^{[k]}$. W is the diagonal matrix such that $W_{ii} = w_i$. 

Whenever we fit a GAM model to a dataset, we choose our link function $g$ and as a result, we know what form the variance function takes. Some common examples include:
* Gaussian: $V(\mu) = 1$
* Poisson: $V(\mu) = \mu$
* Binomial: $V(\mu) = \mu (1 - \mu)$
* Gamma: $V(\mu) = \mu^2$
* Inverse Gaussian: $V(\mu) = \mu^3$

## Conclusion
Today we covered the basis for general additive modelling as an alternative method to general linear models.

## References
1. Wood, S.N. (2017). _Generalized Additive Models_. CRC Press.

‌

## Appendix
**Influence Matrix**: The influence matrix (or hat matrix) of a linear model is the matrix that yields the fitted value vector $\boldsymbol{\hat{\mu}}$ when the post




## Fitting a GAM

Now suppose that for every response variable $y_i$ there exists predictive variables $x_i$ for $i \in \{1, ..., m\}$ . In this situation, minimising the objective function with the least squares objective function becomes computationally difficult. As a result, a new set of methods are needed. This leads to the backfitting algorithm that was invented by Leo Breiman and Jerome Friedman along with the GAMs.

The basic idea behind backfitting is to estimate each smooth component of the additive model by iteratively smoothing the partial residuals from the additive model with respect to the predictive variables. This is done by using the partial residuals relating to the jth smoothing term and then subtracting all the current model term estimates from the response variable except for the jth smoothing term. This is explained better with math.

Given we are trying to estimate the smooth functions for the additive model:
$$y_i = \alpha + \sum_{j=1}^m f_j(x_{ji}) + \epsilon_i.$$
The backfitting algorithm goes as follows:
1. Set $\hat{\alpha} = \bar{y}$ and $\hat{f}_j = 0$ for $j=1, ..., m$.
2. Repeat steps 3 to 5 until $\hat{f}_j$ stops changing.
3. For $j=1, ..., m$  repeat steps 4 and 5.
4. Calculate the partial residuals:
$$e_p^j = \textbf{y} - \hat{\alpha} -\sum_{k \neq j}\hat{f}_k$$
5. Set $\hat{f}_j$ equal to the result of smoothing $e_p^j$ with respect to $x_j$.

The smoothing in step 5 is typically chosen to be a cubic spline smoother but it can also take the form of local polynomial regression, kernel smoothing methods, and other more complex methods.

This will reduce the problem from estimating the degree of smoothness of a model to estimating the smoothing parameter $\lambda$.


---
title: Gaussian process classification and its approximate inference approaches

# # Summary for listings and search engines
# summary: Welcome 👋 We know that first impressions are important, so we've populated your new site with some initial content to help you get familiar with everything in no time.

# Link this post with a project
projects: []

# Date published
date: '2022-12-13T00:00:00Z'

# Date updated
lastmod: '2022-12-13T00:00:00Z'

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption: 
  focal_point: ''
  placement: 2
  preview_only: false

authors:
  - admin

---


## Introduction

In a previous article, we briefly explained the application of Gaussian processes to regression problems. Apart from regression, classification is another important type of problem. Both regression and classification problems can be categorized as 'finding a function mapping from input {{< math >}}$x${{< /math >}} to output {{< math >}}$y${{< /math >}}'. However, compared to regression problems, Gaussian processes encounter many challenging issues when dealing with classification problems. This article will elucidate these issues and provide corresponding solutions.


## 1. Review: Gaussian Process Regression
A Gaussian process is a stochastic process where any point {{< math >}}$x\in R^d${{< /math >}} is assigned a random variable {{< math >}}$f${{< /math >}} and the joint distribution of these variables follows a Gaussian distribution {{< math >}}$f|x\sim\mathcal{N}(\mu,K)${{< /math >}}. Gaussian process is a prior over functions, whose shape (smoothness, etc.) is defined by the mean function {{< math >}}$\mu${{< /math >}} and the covariance {{< math >}}$K=k(X,X)${{< /math >}} where k is a parameterized kernel function. For {{< math >}}$\mu${{< /math >}}, we generally set it to 0 (ie. {{< math >}}$\mu(\,\cdot\,)=0${{< /math >}}). Given a set of input values {{< math >}}$X${{< /math >}} and their corresponding noisy observations {{< math >}}$y${{< /math >}}, we want to predict the function value {{< math >}}$f^*${{< /math >}} at the new point {{< math >}}$x^*${{< /math >}}. The joint distribution of the observed values
{{< math >}}$y${{< /math >}} and the predicted value {{< math >}}$f^*${{< /math >}} is a Gaussian distribution, which has the following form:
{{< math >}}
$$
y,f^*|X,x^*\sim\mathcal{N}(\begin{bmatrix}             y\\            f^*\\          \end{bmatrix}|\,0,\begin{bmatrix}             K_y&k_*\\            k_*&k_{**}\\          \end{bmatrix})\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,(\text{Recap 1.})
$$
{{< /math >}}
{{< math >}}$K_y=K+\sigma_y^2I${{< /math >}}, {{< math >}}$k_*=k(X,x_*)${{< /math >}}, and {{< math >}}$k_{**}=k(x^*,x^*)${{< /math >}}. {{< math >}}$\sigma_y^*${{< /math >}} is used to model the noise in the observed values. By applying Bayes' theorem to the joint distribution above, we obtain the predictive distribution for {{< math >}}$f^*${{< /math >}}: 
{{< math >}}
$$
f^*|x_*,X,y\sim\mathcal{N}(f^*|\mu_*,\Sigma_*)
$$
{{< /math >}}
{{< math >}}
$$
\text{where}\,\,\mu_*=k_*^TK_y^{-1}y\,\,,\,\Sigma_*=k_{**}-k_*^TK_y^{-1}k_*\,\,\,\,\,\,\,\,\,\,\,\,(\text{Recap 2.})
$$
{{< /math >}}
You can check the [previous article](https://yyimingucl.github.io/post/gpr/) for more information.


## 2. Binary Classifcation Problem
In classification problems, our target variable {{< math >}}$y${{< /math >}} is no longer continuous, but rather discrete: {{< math >}}$y = \{+1, -1\}${{< /math >}}. Clearly, we can no longer assume that {{< math >}}$y|x${{< /math >}} follows a Gaussian distribution as in regression problems. A suitable choice is the Bernoulli distribution {{< math >}}$y|x \sim Bernoulli(\theta): p(y=+1|x) = \theta, p(y=-1|x) = 1-\theta${{< /math >}}, where {{< math >}}$\theta \in [0,1]${{< /math >}}. In fact, once we have a good estimate for {{< math >}}$\theta${{< /math >}}, the classification problem is solved (from a discriminative point of view. Another way of looking at classification problems, called the generative perspective, aims to estimate the joint distribution of {{< math >}}$y${{< /math >}} and {{< math >}}$x${{< /math >}}, which is not discussed here). But how can we estimate {{< math >}}$\theta${{< /math >}}? There are many methods for estimating {{< math >}}$\theta${{< /math >}}, such as linear models (logistic regression), neural networks (convolutional neural networks in image recognition), etc. Our title is Gaussian process classification, so naturally we will discuss how to use Gaussian processes to estimate {{< math >}}$\theta${{< /math >}}.

Can we directly treat {{< math >}}$\theta${{< /math >}} as a regression variable ({{< math >}}$y${{< /math >}}) and use a Gaussian process to obtain {{< math >}}$\theta|x \sim\mathcal{N}(\mu, K)${{< /math >}}. The answer is obviously no. There are two reasons for this: (1) the range of {{< math >}}$\theta${{< /math >}} obtained from the Gaussian process is {{< math >}}$(-\infty,\infty)${{< /math >}}, which does not satisfy the requirement that {{< math >}}$\theta\in[0,1]${{< /math >}}, and (2) although we want to estimate {{< math >}}$\theta${{< /math >}}, we cannot directly observe {{< math >}}$\theta${{< /math >}}. We can only observe {{< math >}}$y${{< /math >}} produced by {{< math >}}$\theta${{< /math >}}. To address the first issue, we can use a response function {{< math >}}$\sigma(\,\cdot\,)${{< /math >}} to compress the results obtained from the Gaussian process into the {{< math >}}$[0,1]${{< /math >}} range. Common response functions include the logistic function and the cumulative probability function of the standard Gaussian distribution (probit function). The figure below shows these two functions, as well as the compressed Gaussian process prior:
![png](response_function.png)

## 3. Gaussian Process Classification (GPC)
For distinctions, we denote the regression variable of the Gaussian process as the (latent) variable {{< math >}}$f: f|x \sim \mathcal{N}(\mu, K)${{< /math >}}. With the response function {{< math >}}$\sigma${{< /math >}} introduced in the previous paragraph, we can obtain the likelihood function. For a sample {{< math >}}$(xi, yi)${{< /math >}}, their likelihood is given by: 
{{< math >}}
$$
\begin{equation} p(y_i|x_i,f_i)=\left\{ \begin{array}{ll}       \sigma(f_i(x_i)), & y_i=+1 \\       1-\sigma(f_i(x_i)), & y_i=-1 \\ \end{array}  \right.  \end{equation}
$$
{{< /math >}}
Due to the symmetry of the response function: {{< math >}}$\sigma(-z)=1-\sigma(z)${{< /math >}}, the likelihood can be expressed more concisely as {{< math >}}$p(y_i|x_i,f_i)=\sigma(y_if_i(x_i))${{< /math >}}. It's interesting that for the latent variable f, we don't observe its value (only observe input {{< math >}}${x_i}_i${{< /math >}} and target values {{< math >}}${y_i}_i${{< /math >}}) and we are not interested in it at all. The existence of {{< math >}}$f${{< /math >}} is only for the convenience of modeling discrete y and making the model structure clearer. What we are really interested in is {{< math >}}$\pi(x) = p(y=1|x)${{< /math >}}, especially for new input {{< math >}}$x^*${{< /math >}}, and note that {{< math >}}$\pi(x)${{< /math >}} no longer depends on {{< math >}}$f${{< /math >}}. So **how do we remove this dependence?**

Given the sample {{< math >}}$\{X,y\}${{< /math >}}, the prediction distribution for a new input {{< math >}}$x^*${{< /math >}} can be expressed as:
{{< math >}}
$$
\pi(x^*)=p(y^*=1|X,y,x^*)=\int\sigma(f^*)p(f^*|X,y,x^*)\,df^*\,\,\,\,\,\,\,\,(1)
$$
{{< /math >}}
{{< math >}}
$$
p(f^*|X,y,x^*)=\int p(f^*|X,x^*,f)p(f|X,y)\,df\,\,\,\,\,\,\,\,\,\,\,\,(2)
$$
{{< /math >}}
{{< math >}}$p(f|X,y)=\frac{p(y|f)p(f|X)}{p(y|X)}${{< /math >}} is the posterior distribution of latent variable {{< math >}}$f${{< /math >}}. If we want to solve (1), there are two tricky problems: 1. The posterior distribution of the latent variable {{< math >}}$f, p(f|X,y)${{< /math >}}, is no longer a Gaussian distribution (where {{< math >}}$p(y|f) = \sigma(yf)${{< /math >}} is a non-Gaussian likelihood). 2. The non-linear function {{< math >}}$\sigma${{< /math >}} applied to {{< math >}}$f^*, \sigma(f^*)${{< /math >}}. These two issues make the integration in (1) no longer have a closed-form solution like in regression problems. Approximation is inevitable in this case. This leads us to our second question: **how to approximate the integration in (1)?** Two commonly used methods are given below: 1. Laplacian approximation and 2. Expectation propagation. 

## 4. Laplacian approximation
### 4.1 Introduction
The idea of Laplacian approximation is simple: approximate an unknown distribution {{< math >}}$p${{< /math >}} using a Gaussian distribution {{< math >}}$q${{< /math >}}. The question is, **how do we determine the parameters {{< math >}}$\mu${{< /math >}} and {{< math >}}$\Sigma${{< /math >}} of the Gaussian distribution {{< math >}}$q${{< /math >}}?** Let's start by introducing Laplace's method briefly. Suppose we know that a function {{< math >}}$g(x)${{< /math >}} attains its maximum at {{< math >}}$x_0${{< /math >}}, and we want to evaluate the integral {{< math >}}$\int_a^b g(x)dx${{< /math >}}.
{{< math >}}
$$
\begin{split} 
&\text{Firstly, we define} h(x)=\log(g(x))\\ 
&\Rightarrow \int_a^bg(x)\,dx = \int_a^b\exp(h(x))\,dx\\ 
&\text{Take Second order Taylor expansion of} h(x) \text{at} x_0}\\ 
&\Rightarrow\int_a^b \exp(h(x_0)+h'(x_0)(x-x_0)+\frac{1}{2}h''(x_0)(x-x_0)^2)\,dx\\ 
&\text{we know} g(x) \text{will be maximum at} x_0\\ 
&\text{and} h(x) \text{will also be maximum at} x_0 \Rightarrow h'(x_0)=0\\ 
&\Rightarrow\int_a^bg(x)\,dx\approx\\ 
&\exp(h(x_0))\sqrt{2\pi h''(x_0)}\int_a^b\underbrace{\frac{1}{\sqrt{2\pi h''(x_0)}}\exp(\frac{1}{2}h''(x_0)(x-x_0)^2)}_{\mathcal{N}(x_0,h''(x_0))}\,dx\\ 
&\Rightarrow \text{we only need to find} x_0 \text{and compute} h''(x_0)\\
&\text{then we can get the approximate of desired integral}  
\end{split}
$$
{{< /math >}}

### 4.2 Posterior distribution {{< math >}}$p(f|X,y)${{< /math >}}
Roughly speaking, under the second-order Taylor approximation, {{< math >}}$g(x)${{< /math >}} is proportional to {{< math >}}$\mathcal{N}(x_0, h^{''}(x_0))${{< /math >}}. By replacing {{< math >}}$g(x)${{< /math >}} with the posterior {{< math >}}$p(f|X, y)${{< /math >}} in the above derivation and performing a second-order Taylor expansion around the maximum value {{< math >}}$\hat{f}${{< /math >}}, we can determine the parameters {{< math >}}$\mu${{< /math >}} and {{< math >}}$\Sigma${{< /math >}} of the approximate distribution {{< math >}}$q${{< /math >}}.
{{< math >}}
$$
q(f|X,y)=\mathcal{N}(f|\hat{f},\,A^{-1})\propto \exp{(-\frac{1}{2}(f-\hat{f})^TA(f-\hat{f}))}\,\,\,\,\,\,\,\,\,(3)
$$
{{< /math >}}
(Jumping out of the discussion on this article or Gaussian processes, Laplace's approximation has much broader applications. It is highly recommended that you learn about this method itself. You can refer to this blog post - [Laplace's Method](https://gregorygundersen.com/blog/2019/05/08/laplaces-method/).), where {{< math >}}$\hat{f}=\arg\max_f\,p(f|X,\,y)${{< /math >}}, {{< math >}}$A=-\nabla\nabla\log p(f|X,y)|_{f=\hat{f}}${{< /math >}}. The next question is: **how to find {{< math >}}$\hat{f}${{< /math >}} and its corresponding Hessian matrix {{< math >}}$-\nabla\nabla\log p(\hat{f}|X,y)${{< /math >}}?**
{{< math >}}
$$
\begin{split} \log p(f|X,y)&=\log p(y|f)+\log p(f|X)-\log p(y|X)\\ &\text{take derivative wrt $f$，$\log p(y|X)$ can be viewed as constant}\\ &\propto \log p(y|f)+\log p(f|X)\\ &f|X\sim\mathcal{GP}(\mu(X),K(X,X))\\ &=\log p(y|f) - \frac{1}{2}f^TK^{-1}f-\frac{1}{2}\log|K|-\frac{n}{2}\log2\pi \\ &=\Psi(f)\,\,\,\,\,\,\,\,\,(4) \end{split}
$$
{{< /math >}}
Take derivative of (4) with respect to {{< math >}}$f\Rightarrow${{< /math >}}
{{< math >}}
$$
&\nabla\Psi(f)=\nabla\log p(y|f)-K^{-1}f}\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,(5)
$$
{{< /math >}} 
{{< math >}}
$$
\nabla\nabla\Psi(f)=\nabla\nabla\log p(y|f)-K^{-1}=-W-K^{-1}\,\,\,\,\,\,\,(6) 
$$
{{< /math >}} 
where {{< math >}}$W=-\nabla\nabla\log p(y|f)${{< /math >}} It should be noted that because we assume that each sample {{< math >}}$(x_i, y_i), (x_j, y_j)${{< /math >}} is independent, this means that their corresponding latent variables {{< math >}}$f_i, f_j${{< /math >}} are also independent of each other. So the matrix {{< math >}}$W${{< /math >}} we obtain is a diagonal matrix, which greatly simplifies our calculations. Recall that {{< math >}}$p(y|f) = \sigma(y_i f_i)${{< /math >}}, where {{< math >}}$\sigma(\,\cdot\,)${{< /math >}} has many choices. Below are the logarithmic forms, first-order derivatives, and second-order derivatives of two commonly used functions: the logistic function and the Probit function:
|![png](derivative_of_sigma.png)|
|:--:| 
|*Table from [2]. The first row is for the logistic function and the second is for the probit function.*|
In the case of {{< math >}}$f = \hat{f}${{< /math >}}, our first-order derivative {{< math >}}$(5) = 0 \Rightarrow \hat{f} = K(\nabla \log p(y|\hat{f}))${{< /math >}}. However, {{< math >}}$\nabla \log p(y|\,\cdot\,)${{< /math >}} is nonlinear, and we cannot directly solve it. 

Here, we can use Newton's method for iteration:
{{< math >}}
$$
\begin{split} 
f^{\text{new}}&=f-(\nabla\nabla\Psi)^{-1}\nabla\Psi\\
              &=f+(K^{-1}+W^{-1})^{-1}(\nabla\log p(y|f)-K^{-1}f)\\ 
              &=(K^{-1}+W)^{-1}(Wf+\nabla\log p(y|f)) 
\end{split}
$$
{{< /math >}} 
After finding {{< math >}}$\hat{f}${{< /math >}} and its Hessian matrix (6), we can obtain the posterior distribution of Laplace's approximation:
{{< math >}}
$$
q(f|X,y)=\mathcal{N}(\hat{f},(K^{-1}+W)^{-1})\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,(7)
$$
{{< /math >}}
The biggest drawback of using Laplace's approximation is that the approximate distribution {{< math >}}$q${{< /math >}} we obtain may differ greatly from the true distribution {{< math >}}$p${{< /math >}}. This is because we only use the mean {{< math >}}$\mu${{< /math >}} and covariance {{< math >}}$\Sigma${{< /math >}} of the {{< math >}}$p${{< /math >}} distribution as parameters of the Gaussian distribution {{< math >}}$q${{< /math >}}. However, this information may (1) be far from sufficient to fully describe the distribution {{< math >}}$p${{< /math >}} or (2) not correctly reflect the characteristics of the distribution {{< math >}}$p${{< /math >}}, such as if {{< math >}}$p${{< /math >}} is a long-tail distribution or a multi-modal distribution.
|![png](la_performance.png)|
|:--:| 
|*Laplace's approximation: (left) using Laplace's approximation for a multi-modal distribution p will lead to a very poor approximation q; (right) using Laplace's approximation for a unimodal distribution p will result in a good approximation q.*|

### 4.3 Predication 
By using Laplace's approximation, we can substitute the approximate posterior distribution {{< math >}}$q(f|X,y)${{< /math >}} (equation 7) into the predictive mean (Recap 2) of the regression problem, and obtain the mean of the predictive distribution for a new input {{< math >}}$x^*${{< /math >}}:
{{< math >}}
$$
\mathbb{E}_q(f^*|X,y,x^*)=k_*^TK^{-1}k_*=k_*^T\nabla\log p(y|\hat{f})\,\,\,\,\,\,\,\,\,\,\,(8)
$$
{{< /math >}}
If we carefully observe equation (8), we can express the expectation of the approximate distribution {{< math >}}$q${{< /math >}} as:
{{< math >}}
$$
\mathbb{E}_q(f^*|X,y,x^*)=\sum_{i=1}^nk(x^*,x_i)\nabla\log p(y_i|\hat{f}_i)
$$
{{< /math >}}
Referring to the second column in the table above, we can see that for a positive sample {{< math >}}$(y_i=+1), \nabla\log p(y_i|f^i)\ge0${{< /math >}}. Similarly, a negative sample will result in {{< math >}}$\nabla\log p(y_i|f_i)\le0${{< /math >}}. Simply put, when making the final classification, we determine which class y* belongs to based on the sign of {{< math >}}$E_q${{< /math >}}. If {{< math >}}$x^*${{< /math >}} is very similar to a positive sample {{< math >}}$x_i${{< /math >}} ({{< math >}}$x_i${{< /math >}} is highly likely to belong to the same class as {{< math >}}$x_i\Rightarrow k(x^*,x_i)${{< /math >}}) will be very large {{< math >}}$\Rightarrow k(x^*,x_i)\nabla\log p(y_i|f^i)${{< /math >}} will pull {{< math >}}$E_q${{< /math >}} towards a positive value. Conversely, if it is a negative sample, {{< math >}}$E_q${{< /math >}} will be pulled towards a negative value. The whole principle is very similar to the support vector machine method (kernel), except that we replace the product of the coefficient and label {{< math >}}$c_iy_i${{< /math >}} with {{< math >}}$\nabla\log p(y_i|f_i)${{< /math >}}.

At the same time, we can notice that for a sample {{< math >}}$(x_i,y_i)${{< /math >}} that is easy to classify or can be well-explained by our model, its likelihood will approach 1, {{< math >}}$p(y_i|f_i)\to1${{< /math >}} and its logarithmic likelihood approaches 0, {{< math >}}$\log p(y_i|x_i)\to0${{< /math >}}. {{< math >}}$\Rightarrow k(x^*,x_i)\nabla \log p(y_i|f_i)\to0${{< /math >}}, which will not have a significant impact on predicting {{< math >}}$y^*${{< /math >}}. This is also very similar to the non-support vector in support vector machines!

To classify {{< math >}}$x^*${{< /math >}} only, we can directly use the response function to "squeeze" the {{< math >}}$E_q${{< /math >}} obtained in Equation (8) into the range {{< math >}}$[0,1]${{< /math >}} to obtain the probability of {{< math >}}$y^*=1, \pi(x^*)=\sigma(E_q(f^*|X,y,x^*))${{< /math >}}. Since we are discussing a binary classification problem, we can simply classify as follows:
{{< math >}}
$$
\begin{equation}y^*=\left\{ \begin{array}{ll}       1 & \hat{\pi}(x^*)\ge\frac{1}{2} \\       -1 & \hat{\pi}(x^*)<\frac{1}{2} \\ \end{array}  \right.  \,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,(9)\end{equation}
$$
{{< /math >}}
Since we only calculated the 'maximum' value of the posterior distribution of {{< math >}}$f^*${{< /math >}} (in Gaussian distribution, mean=mode=probability peak value), this method is called 'Maximum a posteriori estimation' (MAP) prediction.

However, sometimes we need to know how 'confident' our prediction is. (For example, in medical applications, if the confidence value is too low, we reject the result). In such cases, we need to calculate the expectation of {{< math >}}$\pi(x^*)${{< /math >}} in formula (1):
{{< math >}}
$$
\bar{\pi}(x^*)=\mathbb{E}_q(\pi(x^*)|X,y,x^*)=\int\sigma(f^*)q(f^*|X,y,x^*)\,df^*\,\,\,\,\,\,\,\,(10)
$$
{{< /math >}}
Recall that {{< math >}}$\pi(x^*) = p(y^*=1|x^*)${{< /math >}}, where {{< math >}}$p${{< /math >}} is the parameter of a Bernoulli distribution. Why can the expectation of {{< math >}}$\pi(x^*)${{< /math >}} reflect the credibility of the result? And what is the difference between it and maximum a posteriori prediction? The blue curve in the following two figures represents the posterior distribution {{< math >}}$q(f^*|X, y, x^*)${{< /math >}}, and the red line represents the maximum a posteriori. In the case of the left figure, the posterior distribution of {{< math >}}$f^*${{< /math >}} has a large variance. When using simple maximum a posteriori prediction, {{< math >}}$\hat{\pi}(x^*) \ge 1/2${{< /math >}}, and it will be classified as {{< math >}}$y^*=+1${{< /math >}}, but this is a very risky behavior because a considerable portion of {{< math >}}$f^*${{< /math >}} is distributed in negative values, so {{< math >}}$\sigma(f^*)\le1/2${{< /math >}} will give {{< math >}}$y^*=-1${{< /math >}}. For this case, Formula (10) considers all possible {{< math >}}$f^*${{< /math >}} and obtains {{< math >}}$\pi(x^*)${{< /math >}} that is much smaller than {{< math >}}$\pi(x^*)${{< /math >}}. At this point, we can reject the classification result to avoid serious consequences caused by misclassification. The corresponding situation on the right figure is that {{< math >}}$\pi(x^*)${{< /math >}} is approximately equal to {{< math >}}$\pi(x^*)${{< /math >}}, and we can confidently use the MAP prediction result.
|![png](MAP_unreliable.png)|
|:--:| 
|*(Left) Unreliable situation of MAP prediction, (Right) Reliable situation of MAP prediction.*|
From the graph, we can see that the variance of the posterior distribution is the key factor that differentiates equation (8) and equation (10). We can calculate the variance of the posterior distribution, {{< math >}}$\mathbb{V}_q(f^*|X,y,x^*)${{< /math >}}, to determine whether to use equation (10). Similarly, due to the existence of {{< math >}}$\sigma(f^*)${{< /math >}}, equation (10) also needs to be approximated or calculated using sampling methods. The calculation of {{< math >}}$\mathbb{V}_q(f^*|X,y,x^*)${{< /math >}} and the computation of equation (10) are both detailed in [2] 3.4.2, so we will not go into further detail here.

## 5. Expectation Propagation - EP
* In this derivation, the response function {{< math >}}$\sigma${{< /math >}} used is the cumulative probability function {{< math >}}$\Phi${{< /math >}} (probit function) of the standard normal distribution: {{< math >}}$p(y_i|f_i) = \Phi(f_i y_i)${{< /math >}}
### 5.1 Introduction 
Compared to the global approximation of Laplace's method, Expectation Propagation (EP) updates the global approximation by iterating through and updating each local approximation until convergence. First, we decompose the likelihood:
{{< math >}}
$$
p(f|X,y)=\frac{1}{Z}p(f|X)\prod_{i=1}^np(y_i|f_i)
$$
{{< /math >}}
{{< math >}}
$$
Z=p(y|X)=\int p(f|X)\prod_{i=1}^np(y_i|f_i)\,df
$$
{{< /math >}}
For the non-Gaussian likelihood of each sample, we make the following approximation:
{{< math >}}
$$
p(y_i|f_i) \simeq t_i(f_i|\tilde{Z}_i,\tilde{\mu}_i,\tilde{\sigma}^2_i)=\tilde{Z}_i\mathcal{N}(f_i|\tilde{\mu}_i,\tilde{\sigma}_i^2)\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,(11)
$$
{{< /math >}}
where {{< math >}}$\tilde{Z}_i,\tilde{\mu}_i,\tilde{\sigma}^2_i${{< /math >}} is called the site parameters for approximating the likelihood of sample {{< math >}}$i${{< /math >}}, it is interesting to note that we are using an unnormalized Gaussian distribution of fi to approximate a Gaussian distribution of {{< math >}}$y_i${{< /math >}}. Upon careful consideration, this approximation is reasonable. We can consider the likelihood {{< math >}}$p(y_i|f_i)${{< /math >}} as a conditional probability distribution of {{< math >}}$y_i${{< /math >}} given {{< math >}}$f_i${{< /math >}}. Since {{< math >}}$y_i${{< /math >}} is fixed, we are more concerned about {{< math >}}$f_i${{< /math >}} and want to know how {{< math >}}$f_i${{< /math >}} affects or explains our target value {{< math >}}$y_i${{< /math >}}. In other words, if the likelihood is a function of {{< math >}}$f_i${{< /math >}}, we are more interested in how the likelihood changes with {{< math >}}$f_i${{< /math >}}. (In regression problems, the distribution of {{< math >}}$y_i|f_i${{< /math >}} is also determined by the selected distribution of {{< math >}}$f_i${{< /math >}}. The difference is that in regression problems, we can directly calculate the distribution of {{< math >}}$y_i|f_i${{< /math >}}, while in classification problems, an approximation is needed.) Since the approximate likelihood of each sample is Gaussian, we can obtain: {{< math >}}$\prod_{i=1}^nt_i(f_i|\tilde{Z}_i,\tilde{\mu}_i,\tilde{\sigma}^2_i)=\mathcal{N}(\tilde{\mu},\tilde{\Sigma})\prod_i\tilde{Z}_i${{< /math >}} where {{< math >}}$\tilde{\mu}=(\tilde{\mu}_1,...,\tilde{\mu}_n)\,\,\text{and}\,\,\tilde{\Sigma}\,\,\text{is diagnoal}\,\,\tilde{\Sigma}_{ii}=\tilde{\sigma}^2_i${{< /math >}}

Then we will define the approximate distribution {{< math >}}$q(f|X,y)${{< /math >}}:
{{< math >}}
$$
\begin{split} &q(f|X,y)=\frac{1}{Z_{EP}}p(f|X)\prod_{i=1}^nt_i(f_i|\tilde{Z}_i,\tilde{\mu}_i,\tilde{\sigma}^2_i)=\mathcal{N}(\mu,\Sigma)\\ &\text{with}\,\,\mu=\Sigma\tilde{\Sigma}^{-1}\tilde{\mu},\,\,\text{and}\,\,\Sigma=(K^{-1}+\tilde{\Sigma}^{-1})^{-1} \end{split}\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,(12)
$$
{{< /math >}}
It is important to note that {{< math >}}$\tilde{\mu},\tilde{\Sigma}${{< /math >}} are the parameters of the local likelihood approximation, while {{< math >}}$\mu,\Sigma${{< /math >}} are the parameters of the global approximate posterior distribution {{< math >}}$q${{< /math >}}, and {{< math >}}$Z_{EP}${{< /math >}} is the normalization constant obtained from the EP algorithm. After defining all of the above, the core problem that the EP algorithm solves arises: **how should we choose the parameters of the local approximate distribution {{< math >}}$t_i${{< /math >}}: {{< math >}}$\tilde{\mu}_i,\tilde{\sigma}_i, \tilde{Z}_i${{< /math >}}?** The main idea of the entire EP algorithm is to update each {{< math >}}$t_i${{< /math >}} sequentially. In general, the EP method iterates through the following four steps:
1. By removing the likelihood {{< math >}}$t_i${{< /math >}} of the sample {{< math >}}$(x_i,y_i)${{< /math >}}, We compute the marginal distribution of {{< math >}}$f_i${{< /math >}} from the current approximate posterior distribution {{< math >}}$q(f_i|X,y)${{< /math >}}, denoted as {{< math >}}$q_{-i}(f_i)${{< /math >}}. Since it looks like we dig a 'hole' around the sample, the distribution {{< math >}}$q_{-i}(f_i)${{< /math >}} is called 'cavity' distribution. 
2. We replace the approximate likelihood {{< math >}}$t_i${{< /math >}} by the true likelihood {{< math >}}$p(y_i|f_i)${{< /math >}} to fill the 'hole', thereby obtaining a new marginal distribution {{< math >}}$q_{-i}(fi)p(y_i|f_i)${{< /math >}} about {{< math >}}$f_i${{< /math >}}.
3. The marginal distribution obtained in the second step is not a Gaussian distribution, so we need to find an approximate Gaussian distribution {{< math >}}$\hat{q}(f_i)${{< /math >}} for it.
4. In the final step, we calculate the parameters of the approximate likelihood {{< math >}}$t_i (\hat{q}(f_i) = t_iq_{-i}(f_i))${{< /math >}}, where  {{< math >}}$\hat{q}(f_i)${{< /math >}} and {{< math >}}$q_{-i}(f_i)${{< /math >}} are known through steps 1 and 3.

Below we will explain these four steps one by one:

The goal of EP is to optimize the local approximation {{< math >}}$t_i${{< /math >}} one by one based on the existing local approximations {{< math >}}$\{t_j\}_{j\neq i}${{< /math >}}. Consider what information is useful for constructing the posterior distribution {{< math >}}$f_i${{< /math >}}: 1. The prior {{< math >}}$p(f_i|X)${{< /math >}}. 2. The local approximation distribution {{< math >}}$\{t_j\}_{j\neq i}${{< /math >}} for each sample. 3. The true likelihood {{< math >}}$p(y_i|f_i) = \sigma(y_if_i)${{< /math >}} for sample {{< math >}}$i${{< /math >}}. Our goal is to integrate all this information to construct the required approximate distribution. First, we combine the prior with the likelihoods of other samples (except sample {{< math >}}$i${{< /math >}}) to obtain the cavity distribution {{< math >}}$q_{-i}(f_i)${{< /math >}}:
{{< math >}}
$$
q_{-i}(f_i)\propto\int p(f|X)\prod_{j\neq i}t_j(f_j|\tilde{Z}_j,\tilde{\mu}_j,\tilde{\sigma}^2_j)\,df_j\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,(13)
$$
{{< /math >}}
Afterwards, the true likelihood of sample i will be combined with (13). To compute {{< math >}}$q_{-i}(f_i)\,\,(13)${{< /math >}}, we need to take two steps:
1. Marginalize {{< math >}}$f_i${{< /math >}} from the approximate distribution of the overall {{< math >}}$q(f|X,y)${{< /math >}} to obtain {{< math >}}$q(f_i|X,y)${{< /math >}}. There are two points to note here: (1) {{< math >}}$q(f|X,y)${{< /math >}} is a Gaussian distribution, and through the marginal property of the Gaussian distribution, we can easily obtain {{< math >}}$q(f_i|X,y)=\mathcal{N}(f_i|\mu_i,\sigma_i^2)${{< /math >}} where {{< math >}}$\sigma^2=\Sigma_{ii} (\Sigma in (12))${{< /math >}}. (2) The {{< math >}}$q(f_i|X,y)${{< /math >}} obtained here does not remove {{< math >}}$t_i${{< /math >}}
{{< math >}}
$$
q(f_i|X,y)=\int p(f|X)t_i(f_i|\tilde{Z}_i,\tilde{\mu}_i,\tilde{\sigma^2_i})\prod_{j\neq i}t_j(f_j|\tilde{Z}_j,\tilde{\mu}_j,\tilde{\sigma}^2_j)\,df_j
$$
{{< /math >}}
Therefore, we need to divide by {{< math >}}$t_i${{< /math >}} to obtain the cavity distribution {{< math >}}$q_{- i} (f_i)= \frac{q(f_i|X,y)}{t_i}${{< /math >}}. The cavity distribution obtained here is still a Gaussian distribution, and one easy way to understand it is that the product of two Gaussian variables is still a Gaussian distribution ({{< math >}}$q(f_i|X,y) = q_{- i} (f_i)\times t_i${{< /math >}}, we can reverse to obtain {{< math >}}$q_{- i }(f_i)${{< /math >}}) by using known {{< math >}}$q(f_i|X,y),t_i${{< /math >}}. 
{{< math >}}
$$
\begin{split} \Rightarrow &\,\,q_{- i }(f_i)=\mathcal{N}(f_i|\mu_{- i},\sigma_{- i}^2) \\ &\text{where}\,\,\mu_{- i }=\sigma_{- i }^2(\sigma_i^{-2}\mu_i-\tilde{\sigma}_i^{-2}\tilde{\mu}_i)\,,\,\,\text{and}\,\,\sigma_{- i }^2=(\sigma_i^{-2}-\tilde{\sigma}_i^{-2})^{-1}  \end{split}  \,\,\,\,\,\, \,\,\,\,\,\, \,\,\,\,\,\, \,\,\,\,\,\, \,\,\,\,\,\, (14)
$$
{{< /math >}}

2. & 3. Next, we need to incorporate the true likelihood {{< math >}}$p(y_i|f_i)${{< /math >}} into the constructed cavity distribution {{< math >}}$q_i^-(f_i)p(y_i|f_i)${{< /math >}} to approximate {{< math >}}$q(f_i|X,y)${{< /math >}}. However, this product is not a Gaussian distribution, so we need to find a Gaussian approximation for this product:
{{< math >}}
$$
\Rightarrow\hat{q}(f_i)\triangleq \hat{Z}_i\mathcal{N}(\hat{\mu}_i,\hat{\sigma}_i^2)\simeq q_{-i }(f_i)p(y_i|f_i) \,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,(15)
$$
{{< /math >}}
Here, we minimize the KL divergence between the product and the approximate distribution {{< math >}}$\hat{q}(f_i)${{< /math >}} by setting it to be a Gaussian distribution that matches the mean and variance of {{< math >}}$q_{-i }(f_i)p(y_i|f_i)${{< /math >}}. The equation is {{< math >}}$\min_{\hat{q}}KL( q_{-i }(f_i)p(y_i|f_i) \|\hat{q}(f_i))${{< /math >}}. After matching the mean and variance, we need to normalize {{< math >}}$\hat{q}(f_i)${{< /math >}} by a constant {{< math >}}$\hat{Z}_i${{< /math >}} to match it with  {{< math >}}$q_{-i }(f_i)p(y_i|f_i)${{< /math >}}. Finally, we obtain the parameters of {{< math >}}$\hat{q}${{< /math >}}: 
{{< math >}}
$$
\hat{Z}_i=\Phi(z_i),\,\,\,\,\,\hat{\mu}_i=\mu_{-i} + \frac{y_i\sigma^2_{-i}\mathcal{N}(z_i)}{\Phi(z_i)\sqrt{1+\sigma^2_{-i} }},
$$
{{< /math >}}
{{< math >}}
$$
\hat{\sigma_i^2}=\sigma^2_{-i} -\frac{\sigma^4_{-i}\mathcal{N}(z_i) }{(1+\sigma_{-i}^2)\Phi(z_i)}(z_i+\frac{\mathcal{N}(z_i)}{\Phi(z_i)}) 
$$
{{< /math >}}
where {{< math >}}$z_i=\frac{y_i\mu_{-i}}{\sqrt{1+\sigma_{-i}^2}}${{< /math >}} and {{< math >}}$\Phi${{< /math >}} is the cumulative distribution function of the standard normal distribution. The complete derivation can be found in section 3.9 [2].

4. Finally, we need to calculate the parameters of the local approximation distribution {{< math >}}$t_i${{< /math >}}:  {{< math >}}$\tilde{Z}_i,\tilde{μ}_i,\tilde{σ}_i^2${{< /math >}}. We replace {{< math >}}$q(f_i|X,y)${{< /math >}} with the results obtained from the previous two steps {{< math >}}$\Rightarrow\hat{q}(f_i)=t_iq_{-i}(f_i)${{< /math >}}, and all three distributions here are Gaussian. Through simple calculations, we can obtain: 
{{< math >}}
$$
\tilde{Z}_i=\hat{Z}_i\sqrt{2\pi}\sqrt{\sigma^2_{- i}+\tilde{\sigma^2_i}}\exp(\frac{1}{2}(\mu_{-i} -\tilde{\mu}_i)^2/(\sigma^2_{-i}+\tilde{\sigma}_i^2)) 
$$
{{< /math >}}
We have now completed the update of the approximate likelihood {{< math >}}$t_i${{< /math >}} for sample {{< math >}}$i${{< /math >}}. Expectation propagation is iteratively applied, updating each local approximation value in turn. Since the update of each local approximation affects the overall approximation, we need to perform multiple updates for each set of samples until the overall likelihood converges.

### 5.2 Predication
Like Laplace approximation, the EP algorithm also approximates the posterior distribution as a Gaussian distribution. For the derivation of the EP predictive distribution, we only need to replace the posterior distribution obtained by Laplace approximation with the approximate posterior obtained by EP, and the remaining derivation is completely identical. The predictive posterior distribution of latent variable {{< math >}}$f^*${{< /math >}}:
{{< math >}}
$$
\begin{split}
\text{Mean: }\mathbb{E}_q[f^*|X,y,x^*]&=k_*^TK^{-1}\mu\\
                                      &=k_*^TK^{-1}(K^{-1}+\tilde{\Sigma}^{-1})^{-1}\tilde{\mu}\\
                                      &=k_*^T(K+\tilde{\Sigma})^{-1}\tilde{\mu}
\end{split}
\,\,\,\,\,\,\,\,\,\,(17)
$$
{{< /math >}}
{{< math >}}
$$
\text{Variance: }\mathbb{V}_q[f^*|X,y,x^*]=k(x^*,x^*)-k_*(K+\tilde{\Sigma})^{-1}k_*\,\,\,\,\,\,\,\,\,\,(18)
$$
{{< /math >}}
From equations (17) and (18), we can obtain the predicted probabilities for binary classification:
{{< math >}}
$$
q(y^*=1|X,y,x^*)=\mathbb{E}_q(\pi^*|X,y,x^*)=\int\Phi(f^*)q(f^*|X,y,x^*)\,\,df^*
$$
{{< /math >}}
where {{< math >}}$q(f^*|X,y,x^*)${{< /math >}} is the predictive distribution of latent variable {{< math >}}$f^*${{< /math >}} with mean (17) and variance (18). By computing the integral on the right-hand side of the equation, we obtain the final required predictive probability: {{< math >}}$q(y^*=1|X,y,x^*)=\Phi(\frac{k_*(K+\tilde{\Sigma})^{-1}\tilde{\mu}}{\sqrt{1+k(x^*,x^*)-k_*(K+\tilde{\Sigma})^{-1}k_*}})${{< /math >}}

## Conclusion 
Actually, Gaussian process classification simply replaces the linear model in logistic regression with a Gaussian process. The difficulty lies in the fact that the Gaussian distribution assumption for the target variable y is not valid. The posterior distribution no longer has a closed-form solution, as in the regression problem, and an appropriate approximation is required. The article briefly introduces two commonly used approximation methods: Laplace approximation and expectation propagation. The Laplace method approximates the posterior distribution globally as a Gaussian distribution, which has the advantage of being simple and fast to compute, but has the disadvantage of being very poor in approximating multi-modal posterior distributions. Expectation propagation iteratively updates the posterior distribution of each sample locally, which has the advantage of relatively good approximation performance for any posterior distribution, but the disadvantage of being computationally complex and cannot guarantee the convergence of the global approximation.

Stepping out of this article or Gaussian process discussion, Laplace approximation has a wider range of applications, and I highly recommend everyone to learn about this method itself. You can refer to this blog post - [Laplace's Method](https://gregorygundersen.com/blog/2019/05/08/laplaces-method/). Similarly, expectation propagation is not only applied in the approximation of Gaussian process classification, but also provides a good idea for the approximation of posterior distributions, and plays a very important role in variational inference.

- Reference:
  * [1] MacKay, David JC. "Introduction to Gaussian processes."NATO ASI series F computer and systems sciences168 (1998): 133-166.
  * [2] Williams, Christopher KI, and Carl Edward Rasmussen.Gaussian processes for machine learning. Vol. 2. No. 3. Cambridge, MA: MIT press, 2006.
  * [3] Minka T P. Expectation propagation for approximate Bayesian inference[J]. arXiv preprint arXiv:1301.2294, 2013.

<!-- ### [❤️ Click here to become a sponsor and help support Wowchemy's future ❤️](https://wowchemy.com/sponsor/)

As a token of appreciation for sponsoring, you can **unlock [these](https://wowchemy.com/sponsor/) awesome rewards and extra features 🦄✨**

## Ecosystem

- **[Hugo Academic CLI](https://github.com/wowchemy/hugo-academic-cli):** Automatically import publications from BibTeX

## Inspiration

[Check out the latest **demo**](https://academic-demo.netlify.com/) of what you'll get in less than 10 minutes, or [view the **showcase**](https://wowchemy.com/user-stories/) of personal, project, and business sites.

## Features

- **Page builder** - Create _anything_ with [**widgets**](https://wowchemy.com/docs/page-builder/) and [**elements**](https://wowchemy.com/docs/content/writing-markdown-latex/)
- **Edit any type of content** - Blog posts, publications, talks, slides, projects, and more!
- **Create content** in [**Markdown**](https://wowchemy.com/docs/content/writing-markdown-latex/), [**Jupyter**](https://wowchemy.com/docs/import/jupyter/), or [**RStudio**](https://wowchemy.com/docs/install-locally/)
- **Plugin System** - Fully customizable [**color** and **font themes**](https://wowchemy.com/docs/customization/)
- **Display Code and Math** - Code highlighting and [LaTeX math](https://en.wikibooks.org/wiki/LaTeX/Mathematics) supported
- **Integrations** - [Google Analytics](https://analytics.google.com), [Disqus commenting](https://disqus.com), Maps, Contact Forms, and more!
- **Beautiful Site** - Simple and refreshing one page design
- **Industry-Leading SEO** - Help get your website found on search engines and social media
- **Media Galleries** - Display your images and videos with captions in a customizable gallery
- **Mobile Friendly** - Look amazing on every screen with a mobile friendly version of your site
- **Multi-language** - 34+ language packs including English, 中文, and Português
- **Multi-user** - Each author gets their own profile page
- **Privacy Pack** - Assists with GDPR
- **Stand Out** - Bring your site to life with animation, parallax backgrounds, and scroll effects
- **One-Click Deployment** - No servers. No databases. Only files.

## Themes

Wowchemy and its templates come with **automatic day (light) and night (dark) mode** built-in. Alternatively, visitors can choose their preferred mode - click the moon icon in the top right of the [Demo](https://academic-demo.netlify.com/) to see it in action! Day/night mode can also be disabled by the site admin in `params.toml`.

[Choose a stunning **theme** and **font**](https://wowchemy.com/docs/customization) for your site. Themes are fully customizable.

## License

Copyright 2016-present [George Cushen](https://georgecushen.com).

Released under the [MIT](https://github.com/wowchemy/wowchemy-hugo-themes/blob/master/LICENSE.md) license. -->

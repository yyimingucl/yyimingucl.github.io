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
A Gaussian process is a stochastic process where any point {{< math >}}$x\in R^d${{< /math >}} is assigned a random variable {{< math >}}$f${{< /math >}} and the joint distribution of these variables follows a Gaussian distribution {{< math >}}$f|x\sim/mathcal{N}(\mu,K)${{< /math >}}. Gaussian process is a prior over functions, whose shape (smoothness, etc.) is defined by the mean function {{< math >}}$\mu${{< /math >}} and the covariance {{< math >}}$K=k(X,X)${{< /math >}} where k is a parameterized kernel function. For {{< math >}}$\mu${{< /math >}}, we generally set it to 0 (ie. {{< math >}}$\mu(\,\cdot\,)=0${{< /math >}}). Given a set of input values {{< math >}}$X${{< /math >}} and their corresponding noisy observations {{< math >}}$y${{< /math >}}, we want to predict the function value {{< math >}}$f^*${{< /math >}} at the new point {{< math >}}$x^*${{< /math >}}. The joint distribution of the observed values
{{< math >}}$y${{< /math >}} and the predicted value {{< math >}}$f^*${{< /math >}} is a Gaussian distribution, which has the following form:
{{< math >}}
$$
y,f^*|X,x^*\sim/mathcal{N}(\begin{bmatrix}             y\\            f^*\\          \end{bmatrix}|\,0,\begin{bmatrix}             K_y&k_*\\            k_*&k_{**}\\          \end{bmatrix})\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,\,(\text{Recap 1.})
$$
{{< /math >}}
{{< math >}}$K_y=K+\sigma_y^2I${{< /math >}}, {{< math >}}$k_*=/mathcal{k}(X,x_*)${{< /math >}}, and {{< math >}}$k_{**}=/mathcal{k}(x^*,\bm{x}^*)${{< /math >}}. {{< math >}}$\sigma_y^*${{< /math >}} is used to model the noise in the observed values. By applying Bayes' theorem to the joint distribution above, we obtain the predictive distribution for {{< math >}}$f^*${{< /math >}}: 
{{< math >}}
$$
f^*|x_*,X,y\sim/mathcal{N}(f^*|\mu_*,\Sigma_*),\,\,\,\,\mu_*=k_*^TK_y^{-1}y\,\,,\,\Sigma_*=k_{**}-k_*^TK_y^{-1}k_*\,\,\,\,\,\,\,\,\,\,\,\,(\text{Recap 2.})
$$
{{< /math >}}
You can check the [previous article](https://yyimingucl.github.io/post/gpr/) for more information.


## 2. Binary Classifcation Problem
In classification problems, our target variable {{< math >}}$y${{< /math >}} is no longer continuous, but rather discrete: {{< math >}}$y = \{+1, -1\}${{< /math >}}. Clearly, we can no longer assume that {{< math >}}$y|x${{< /math >}} follows a Gaussian distribution as in regression problems. A suitable choice is the Bernoulli distribution {{< math >}}$y|x \sim Bernoulli(\theta): p(y=+1|x) = \theta, p(y=-1|x) = 1-\theta${{< /math >}}, where {{< math >}}$\theta \in [0,1]${{< /math >}}. In fact, once we have a good estimate for {{< math >}}$\theta${{< /math >}}, the classification problem is solved (from a discriminative point of view. Another way of looking at classification problems, called the generative perspective, aims to estimate the joint distribution of {{< math >}}$y${{< /math >}} and {{< math >}}$x${{< /math >}}, which is not discussed here). But how can we estimate {{< math >}}$\theta${{< /math >}}? There are many methods for estimating {{< math >}}$\theta${{< /math >}}, such as linear models (logistic regression), neural networks (convolutional neural networks in image recognition), etc. Our title is Gaussian process classification, so naturally we will discuss how to use Gaussian processes to estimate {{< math >}}$\theta${{< /math >}}.

Can we directly treat {{< math >}}$\theta${{< /math >}} as a regression variable ({{< math >}}$y${{< /math >}}) and use a Gaussian process to obtain {{< math >}}$\theta|x ~ /mathcal{N}(\mu, K)${{< /math >}}. The answer is obviously no. There are two reasons for this: (1) the range of {{< math >}}$\theta${{< /math >}} obtained from the Gaussian process is {{< math >}}$(-\infty,\infty)${{< /math >}}, which does not satisfy the requirement that {{< math >}}$\theta\in[0,1]${{< /math >}}, and (2) although we want to estimate {{< math >}}$\theta${{< /math >}}, we cannot directly observe {{< math >}}$\theta${{< /math >}}. We can only observe {{< math >}}$y${{< /math >}} produced by {{< math >}}$\theta${{< /math >}}. To address the first issue, we can use a response function {{< math >}}$\sigma(\,\cdot\,)${{< /math >}} to compress the results obtained from the Gaussian process into the {{< math >}}$[0,1]${{< /math >}} range. Common response functions include the logistic function and the cumulative probability function of the standard Gaussian distribution (probit function). The figure below shows these two functions, as well as the compressed Gaussian process prior:
![png](response_function.png)

## 3. Gaussian Process Classification (GPC)
For distinctions, we denote the regression variable of the Gaussian process as the (latent) variable {{< math >}}$f: f|x \sim /mathcal{N}(\mu, K)${{< /math >}}. With the response function {{< math >}}$\sigma${{< /math >}} introduced in the previous paragraph, we can obtain the likelihood function. For a sample {{< math >}}$(xi, yi)${{< /math >}}, their likelihood is given by: 
{{< math >}}
$$
\begin{equation} p(y_i|x_i,f_i)=\left\{ \begin{array}{ll}       \sigma(f_i(x_i)), & y_i=+1 \\       1-\sigma(f_i(x_i)), & y_i=-1 \\ \end{array}  \right.  \end{equation}
$$
{{< /math >}}
Due to the symmetry of the response function: {{< math >}}$\sigma(-z)=1-\sigma(z)${{< /math >}}, the likelihood can be expressed more concisely as {{< math >}}$p(y_i|x_i,f_i)=\sigma(y_if_i(x_i))${{< /math >}}. It's interesting that for the latent variable f, we don't observe its value (only observe input {{< math >}}${x_i}_i${{< /math >}} and target values {{< math >}}${y_i}_i${{< /math >}}) and we are not interested in it at all. The existence of {{< math >}}$f${{< /math >}} is only for the convenience of modeling discrete y and making the model structure clearer. What we are really interested in is {{< math >}}$\pi(x) = p(y=1|x)${{< /math >}}, especially for new input {{< math >}}$x^*${{< /math >}}, and note that {{< math >}}$\pi(x)${{< /math >}} no longer depends on {{< math >}}$f${{< /math >}}. So **how do we remove this dependence?**

Given the sample {{< math >}}$\{X,y\}${{< /math >}}, the prediction distribution for a new input {{< math >}}$x^*${{< /math >}} can be expressed as:
{{< math >}}
$$
\pi(x^*)=p(y^*=1|X,y,x^*)=\int\sigma(f^*)\underbrace{p(f^*|X,y,x^*)}_{The predictive distributio no latent variable f^* }\,df^*\,\,\,\,\,\,\,\,(1)
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
\begin{split} &\text{Firstly, we define $h(x)=\log(g(x))$}\\ &\Rightarrow \int_a^bg(x)\,dx = \int_a^b\exp(h(x))\,dx\\ &\text{在 $x_0$ 对 $h(x)$ Take Second order Taylor expansion}\\ &\Rightarrow\int_a^b \exp(h(x_0)+h'(x_0)(x-x_0)+\frac{1}{2}h''(x_0)(x-x_0)^2)\,dx\\ &\text{we know $g(x)$ will be maximum at $x_0$ $\Rightarrow h(x)$ will also be maximum at $x_0$ $\Rightarrow h'(x_0)=0$}\\ &\Rightarrow\int_a^bg(x)\,dx\approx \exp(h(x_0))\sqrt{2\pi h''(x_0)}\int_a^b\underbrace{\frac{1}{\sqrt{2\pi h''(x_0)}}\exp(\frac{1}{2}h''(x_0)(x-x_0)^2)}_{\mathcal{N}(x_0,h''(x_0))}\,dx\\ &\Rightarrow \text{we only need to find $x_0$ and compute $h''(x_0)$, then we can get the approximate of desired integral}  \end{split}
$$
{{< /math >}}
### 4.2 Posterior distribution {{< math >}}$p(f|X,y)${{< /math >}}




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

---
layout: post
title:  "How to do a Coarsened Exact Matching"
date:   2019-04-14 16:42:00
excerpt: "Coarsened Exact Matching (CEM) 方法由University of Milan的Stefano M. 
          Iacus，Harvard University的Gary King， 以及University of Trieste的Giuseppe Porro提出"
author: Chi Shen
tags: [R, CEM, Matching]
feature: https://img.huffingtonpost.com/asset/578782b91a00002700dd151c.jpeg
comments: true
---

### Chi Shen, April 14 2019

<br>

> "The goal of matching is to reduce imbalance in the empirical
> distribution of the pre-treatment confounders between the treated and
> control groups."

> <p align = "right">----- Stuart, Elizabeth A. (2010) </p>

------------------------------------------------------------------------

<br>

## 1. 背景介绍
-----------

Coarsened Exact Matching (CEM) 方法由University of Milan的Stefano M.
Iacus，Harvard University的Gary King， 以及University of
Trieste的Giuseppe Porro提出，其算法最早于2008年在线发表在Gary
King的Harvard University主页上 [“Matching for Causal Inference Without
Balance Checking.”](http://gking.harvard.edu/files/abs/cem-abs.shtml)。

其后分别在 **Journal of Statistical Software (2009)** 和 **The Stata
Journal (2009)** 上发表了R和Stata版本的相关package，
其正式成果于2011年发表于 **Journal of the American Statistical
Association** 上，以及2012的[Political
Analysis](http://j.mp/2nRpUHQ)上。

CEM亦可称之为“Cochran Exact Matching” ，衍生于Cochran于1986年提出的
**subclassification-based method （Cochran, W. G., 1968)**，
在2011年发表的论文中Gary King等人亦将CEM与PSM (Propensity Score
Matching)进行了比较，提出了CEM的优势。

<br>

## 2. PSM与CEM
-----------

在CEM方法提出之前，已有较多的匹配方法，其中最具有代表性的就是 **Paul R.
Rosenbaum and Rubin (1983)** 提出的Propensity score matching (PSM)，
截至目前，使用PSM发表的论文已超过10000篇，是目前最常用的匹配方法。从下图中也能看出两种方法在发表论文中使用的差距。Gary
King在一篇Working paper (“Why Propensity Scores Should Not
Be Used for Matching”)中指出了PSM的不足，原文如下：

> We show here that PSM, as it is most commonly used in practice (or
> with many of the refinements that have been proposed by statisticians
> and methodologists), increases imbalance, inefficiency, model
> dependence, research discretion, and statistical bias at some point in
> both real data and in data generated to meet the requirements of PSM
> theory. <font color="#00dd00">In fact, the more balanced the data, or
> the more balanced it becomes by pruning some observations through
> matching, the more likely PSM will degrade inferences — a problem we
> refer to as the PSM paradox</font>. If one’s data are so imbalanced
> that making valid causal inferences from it without heavy modeling
> assumptions is impossible, then the paradox we identify is avoidable
> and PSM will reduce imbalance but then the data are not very useful
> for causal inference by any method.

也许只有名字里带King的人论文题目敢这么写，当然，不出你所料，自然也会有这样一篇文章“Why
propensity scores should be used for matching” **(Ben Jann, 2017).**
,至于CEM与PSM孰优孰劣，只能依据个人研究自行判断了。

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-04-14-How-to-do-a-Coarsened-Exact-Matching/figure-markdown_strict/Number-of-article-1.png?raw=true)

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-04-14-How-to-do-a-Coarsened-Exact-Matching/figure-markdown_strict/Number-of-article-2.png?raw=true)

<br>

## 3. 算法
-------

CEM没有PSM那么复杂的反事实假设，其算法一共可分为三步：

-   将所有纳入匹配的协变量，粗化（coarsen）成离散分组，对于已是分类变量的协变量，可以自行设定（比如，性别、教育程度等），而对于连续型变量
    CEM算法会根据自动频率分布直方图进行自动粗化，当然也可以自行手动设定cutpoint,s,
    并且可以设定自动粗化的方法(Sturges’ rule, the default), “fd”
    (Freedman-Diaconis’ rule), “scott” (Scott’s rule) and “ss”
    (ShimazakiShinomoto’s rule)。
-   对所有粗化之后的分类进行分层并排序
-   删除未同时包含至少一个Treat组和Control组的层

<br>

## 4. 不平衡度测量
-------------

CEM中引入了一个参数L1来衡量Treat组和Control组之间在协变量上的不平衡度，其计算公式如下：

> *L*<sub>1</sub>(*f*, *g*) = 
> ∑|*f*<sub>*e*</sub>(1...*k*)−*g*<sub>*e*</sub>(1...*k*)|

L1取值在0~1之间，0代表完全平衡，1代表完全不平衡。若L1为0.6，即说明有40%的粗化后各层的频率分布
直方图在Treat组和Control组之间是重叠的，L1即是根据各层的相对频率差值求和而得，示例如下：

    假设有三个协变量（X1...X），粗化后个协变量的分类为（2, 3, 5），那么粗化后共有2*3*5=30个层，
    我们随机为Treat组和Control组在360个层生成一个正整数，最后计算频率并画出直方图。

    library(magrittr)
    # Set seed
    set.seed(2019)

    # Data
    strate <- sample(paste("str", c(1:30), sep = "_"), size = 2000, replace = TRUE)
    group <- rep(c("Treat", "Control"), 1000)

    imb <- data.frame(strate = strate, group = group)
    imb_sum <- prop.table(table(imb$group, imb$strate), 1) %>% as.data.frame()

    # Plot
    ggplot(imb_sum) +
         geom_bar(aes(x = Var2, y = Freq, fill = Var1), position = "dodge", stat = "identity") +
         scale_fill_manual(name = "", values = c(3, 2)) +
         labs(y = "Prop", x = "Strates") +
         theme_chi

![](https://github.com/shumchi/shumchi.github.io/blob/master/_posts/2019-04-14-How-to-do-a-Coarsened-Exact-Matching/figure-markdown_strict/imbalance-1.png?raw=true)

<br>

## 5. 匹配
-------

**采用R语言中的cem包，示例的dataset为cem包中自带的LeLonde**

### 5.1 示例的dataset介绍

**Outcome variable:** re78  
**Treat variable:** treated, 1 = treat group, 0 = control group  
**Control variable:** c("age", "education", "black", "married",
"nodegree", "re74", "re75", "hispanic", "u74", "u75","q1")

    library(cem)
    data(LeLonde)
    df <- LeLonde[c("treated", "re78", "age", "education", "black", "married", "nodegree", "re74", "re75", "hispanic", "u74", "u75", "q1")] %>%
            na.omit()
           
    str(df)

    ## 'data.frame':    650 obs. of  13 variables:
    ##  $ treated  : num  1 0 1 0 0 0 0 1 0 0 ...
    ##  $ re78     : num  11657 499 16717 30248 4394 ...
    ##  $ age      : int  20 39 49 26 38 28 27 33 44 31 ...
    ##  $ education: int  12 12 8 8 10 12 12 12 9 12 ...
    ##  $ black    : int  0 1 0 0 1 0 1 1 1 0 ...
    ##  $ married  : int  1 1 1 1 1 0 1 1 1 0 ...
    ##  $ nodegree : int  0 0 1 1 1 0 0 0 1 0 ...
    ##  $ re74     : num  8644 19785 9715 37212 14759 ...
    ##  $ re75     : num  8644 6608 7286 36941 14702 ...
    ##  $ hispanic : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ u74      : num  0 0 0 0 0 1 1 0 0 1 ...
    ##  $ u75      : num  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ q1       : Factor w/ 6 levels "agree","disagree",..: 5 3 4 4 6 1 4 1 3 3 ...
    ##  - attr(*, "na.action")= 'omit' Named int  1 11 13 17 26 35 49 50 86 98 ...
    ##   ..- attr(*, "names")= chr  "15993" "16003" "16005" "16009" ...

### 5.1 匹配前准备：不平衡度测量

利用 *imbalance()*
函数对匹配的数据的不平衡进行测量，如下结果所示，整体不平衡度为0.902，
意味数据存在较高的不平衡性。

对于连续型变量，默认计算mean in difference，对于分类变量默认计算chi
square。

    imbalance(group = df$treated, data = df[, -c(1, 2)])

    ## 
    ## Multivariate Imbalance Measure: L1=0.902
    ## Percentage of local common support: LCS=5.8%
    ## 
    ## Univariate Imbalance Measures:
    ## 
    ##               statistic   type           L1 min 25%      50%       75%
    ## age        -0.252373042 (diff) 5.102041e-03   0   0   0.0000   -1.0000
    ## education   0.153634710 (diff) 8.463851e-02   1   0   1.0000    1.0000
    ## black      -0.010322734 (diff) 1.032273e-02   0   0   0.0000    0.0000
    ## married    -0.009551495 (diff) 9.551495e-03   0   0   0.0000    0.0000
    ## nodegree   -0.081217371 (diff) 8.121737e-02   0  -1   0.0000    0.0000
    ## re74      -18.160446880 (diff) 5.551115e-17   0   0 284.0715  806.3452
    ## re75      101.501761679 (diff) 5.551115e-17   0   0 485.6310 1238.4114
    ## hispanic   -0.010144756 (diff) 1.014476e-02   0   0   0.0000    0.0000
    ## u74        -0.045582186 (diff) 4.558219e-02   0   0   0.0000    0.0000
    ## u75        -0.065555292 (diff) 6.555529e-02   0   0   0.0000    0.0000
    ## q1          7.494021189 (Chi2) 1.067078e-01  NA  NA       NA        NA
    ##                  max
    ## age          -6.0000
    ## education     1.0000
    ## black         0.0000
    ## married       0.0000
    ## nodegree      0.0000
    ## re74      -2139.0195
    ## re75        490.3945
    ## hispanic      0.0000
    ## u74           0.0000
    ## u75           0.0000
    ## q1                NA

### 5.2 开始匹配

利用 *cem()*
函数进行CEM匹配，参数treatment用来指定分组变量，drop用来排除结局变量。

从结果可以看出，Control组从全部392个样本中匹配上95例，Treat组从全部258个样本中匹配上84例，匹配后样本的整体
L1为0.605，相比匹配前，有所下降。另外，从statistic列的结果也可看出，在各匹配变量中两组之间无统计学差异。

    mat <- cem(treatment = "treated", data = df, drop = "re78", eval.imbalance = TRUE)
    mat

    ##            G0  G1
    ## All       392 258
    ## Matched    95  84
    ## Unmatched 297 174
    ## 
    ## 
    ## Multivariate Imbalance Measure: L1=0.605
    ## Percentage of local common support: LCS=25.8%
    ## 
    ## Univariate Imbalance Measures:
    ## 
    ##               statistic   type           L1 min 25% 50%      75%       max
    ## age        9.404762e-02 (diff) 0.000000e+00   0   0   1   0.0000    0.0000
    ## education -2.222222e-02 (diff) 2.222222e-02   0   0   0   0.0000    0.0000
    ## black      1.110223e-16 (diff) 0.000000e+00   0   0   0   0.0000    0.0000
    ## married    0.000000e+00 (diff) 0.000000e+00   0   0   0   0.0000    0.0000
    ## nodegree   0.000000e+00 (diff) 0.000000e+00   0   0   0   0.0000    0.0000
    ## re74       1.496418e+02 (diff) 0.000000e+00   0   0   0 463.3308  889.5410
    ## re75       1.587521e+02 (diff) 0.000000e+00   0   0   0 843.6863 -640.9307
    ## hispanic   0.000000e+00 (diff) 0.000000e+00   0   0   0   0.0000    0.0000
    ## u74        0.000000e+00 (diff) 0.000000e+00   0   0   0   0.0000    0.0000
    ## u75        0.000000e+00 (diff) 0.000000e+00   0   0   0   0.0000    0.0000
    ## q1         2.083349e+00 (Chi2) 1.387779e-17  NA  NA  NA       NA        NA

**然而**，此处我产生一个小疑惑，纳入匹配变量的数据类型是否会影响粗化分组过程，从而影响匹配结局？

**因此**，我将black、married等分类变量设置成factor类型，比较前后不平衡度测量及匹配结果。

**区别**，比较结果看出，与前述结果并无差异，仅在测量不平衡度时对于分类变量采用了chi
square.  
<font color="#FF0000">另一个明显的区别在于，对于设置成分类变量后，在进行匹配时，不会再对分类变量进行粗化。</font>

    df_1 <- df
    df_1$black <- as.factor(df_1$black)
    df_1$married <- as.factor(df_1$married)
    df_1$nodegree <- as.factor(df_1$nodegree)

    imbalance(group = df$treated, data = df_1[, - c(1, 2)])

    ## 
    ## Multivariate Imbalance Measure: L1=0.902
    ## Percentage of local common support: LCS=5.8%
    ## 
    ## Univariate Imbalance Measures:
    ## 
    ##              statistic   type           L1 min 25%      50%       75%
    ## age        -0.25237304 (diff) 5.102041e-03   0   0   0.0000   -1.0000
    ## education   0.15363471 (diff) 8.463851e-02   1   0   1.0000    1.0000
    ## black       0.04859165 (Chi2) 1.032273e-02  NA  NA       NA        NA
    ## married     0.04724359 (Chi2) 9.551495e-03  NA  NA       NA        NA
    ## nodegree    5.54497304 (Chi2) 8.121737e-02  NA  NA       NA        NA
    ## re74      -18.16044688 (diff) 5.551115e-17   0   0 284.0715  806.3452
    ## re75      101.50176168 (diff) 5.551115e-17   0   0 485.6310 1238.4114
    ## hispanic   -0.01014476 (diff) 1.014476e-02   0   0   0.0000    0.0000
    ## u74        -0.04558219 (diff) 4.558219e-02   0   0   0.0000    0.0000
    ## u75        -0.06555529 (diff) 6.555529e-02   0   0   0.0000    0.0000
    ## q1          7.49402119 (Chi2) 1.067078e-01  NA  NA       NA        NA
    ##                  max
    ## age          -6.0000
    ## education     1.0000
    ## black             NA
    ## married           NA
    ## nodegree          NA
    ## re74      -2139.0195
    ## re75        490.3945
    ## hispanic      0.0000
    ## u74           0.0000
    ## u75           0.0000
    ## q1                NA

    cem(treatment = "treated", data = df_1, drop = "re78", eval.imbalance = TRUE)

    ## Warning in chisq.test(cbind(t1[keep], t2[keep])): Chi-squared approximation
    ## may be incorrect

    ## Warning in chisq.test(cbind(t1[keep], t2[keep])): Chi-squared approximation
    ## may be incorrect

    ##            G0  G1
    ## All       392 258
    ## Matched    95  84
    ## Unmatched 297 174
    ## 
    ## 
    ## Multivariate Imbalance Measure: L1=0.605
    ## Percentage of local common support: LCS=25.8%
    ## 
    ## Univariate Imbalance Measures:
    ## 
    ##               statistic   type           L1 min 25% 50%      75%       max
    ## age        9.404762e-02 (diff) 0.000000e+00   0   0   1   0.0000    0.0000
    ## education -2.222222e-02 (diff) 2.222222e-02   0   0   0   0.0000    0.0000
    ## black      7.452718e-32 (Chi2) 0.000000e+00  NA  NA  NA       NA        NA
    ## married    2.929392e-31 (Chi2) 0.000000e+00  NA  NA  NA       NA        NA
    ## nodegree   7.216872e-31 (Chi2) 0.000000e+00  NA  NA  NA       NA        NA
    ## re74       1.496418e+02 (diff) 0.000000e+00   0   0   0 463.3308  889.5410
    ## re75       1.587521e+02 (diff) 0.000000e+00   0   0   0 843.6863 -640.9307
    ## hispanic   0.000000e+00 (diff) 0.000000e+00   0   0   0   0.0000    0.0000
    ## u74        0.000000e+00 (diff) 0.000000e+00   0   0   0   0.0000    0.0000
    ## u75        0.000000e+00 (diff) 0.000000e+00   0   0   0   0.0000    0.0000
    ## q1         2.083349e+00 (Chi2) 1.387779e-17  NA  NA  NA       NA        NA

### 5.3 匹配后处理

匹配后生成的匹配对象（mat），其类为 cem.match，其属性实为 list。

#### mat的数据结构如下:

其中详细记录了匹配结果与参数，需要关注的有breaks，matched，w三个对象中的信息。

-   breaks为一个list，其中记录匹配变量自动粗化过程中设置的cutpoint（仅包含numeric类型）  
-   matched实为一个逻辑向量，记录了该个体是否进入了匹配后的样本  
-   w为匹配权重（详细见5.5），用于后续的统计分析中

#### 提取匹配后的样本:

**CEM**包中给出了不用单独提取出匹配后样本进行回归的函数 *att()*
,不过我个人比较倾向将匹配后的样本单独存储为一个对象， 但是 **CEM**
包中并未给出像 **MatchIt**中的 *match.out()*
函数，至少我还没有找到，所以只能自己动手，丰衣足食  

    ## List of 21
    ##  $ call          : language cem.match(data = data, verbose = verbose)
    ##  $ strata        : int [1:650] 112 466 489 260 460 335 355 451 483 409 ...
    ##  $ n.strata      : int 491
    ##  $ vars          : chr [1:11] "age" "education" "black" "married" ...
    ##  $ drop          : chr [1:2] "re78" "treated"
    ##  $ breaks        :List of 10
    ##   ..$ age      : num [1:11] 17 20.8 24.6 28.4 32.2 36 39.8 43.6 47.4 51.2 ...
    ##   ..$ education: num [1:11] 3 4.2 5.4 6.6 7.8 9 10.2 11.4 12.6 13.8 ...
    ##   ..$ black    : num [1:11] 0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 ...
    ##   ..$ married  : num [1:11] 0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 ...
    ##   ..$ nodegree : num [1:11] 0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 ...
    ##   ..$ re74     : num [1:11] 0 3957 7914 11871 15828 ...
    ##   ..$ re75     : num [1:11] 0 3743 7486 11229 14973 ...
    ##   ..$ hispanic : num [1:11] 0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 ...
    ##   ..$ u74      : num [1:11] 0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 ...
    ##   ..$ u75      : num [1:11] 0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 ...
    ##  $ treatment     : chr "treated"
    ##  $ n             : int 650
    ##  $ groups        : Factor w/ 2 levels "0","1": 2 1 2 1 1 1 1 2 1 1 ...
    ##  $ g.names       : chr [1:2] "0" "1"
    ##  $ n.groups      : int 2
    ##  $ group.idx     :List of 2
    ##   ..$ G0: int [1:392] 2 4 5 6 7 9 10 11 12 13 ...
    ##   ..$ G1: int [1:258] 1 3 8 14 15 16 18 24 25 26 ...
    ##  $ group.len     : Named int [1:2] 392 258
    ##   ..- attr(*, "names")= chr [1:2] "G0" "G1"
    ##  $ mstrata       : int [1:650] NA NA NA NA NA NA NA NA NA NA ...
    ##  $ mstrataID     : int [1:62] 52 83 86 100 113 137 165 175 177 180 ...
    ##  $ matched       : logi [1:650] FALSE FALSE FALSE FALSE FALSE FALSE ...
    ##  $ baseline.group: chr "1"
    ##  $ tab           : num [1:3, 1:2] 392 95 297 258 84 174
    ##   ..- attr(*, "dimnames")=List of 2
    ##   .. ..$ : chr [1:3] "All" "Matched" "Unmatched"
    ##   .. ..$ : chr [1:2] "G0" "G1"
    ##  $ k2k           : logi FALSE
    ##  $ w             : num [1:650] 0 0 0 0 0 0 0 0 0 0 ...
    ##  $ imbalance     :List of 2
    ##   ..$ tab:'data.frame':  11 obs. of  8 variables:
    ##   .. ..$ statistic: num [1:11] 9.40e-02 -2.22e-02 1.11e-16 0.00 0.00 ...
    ##   .. ..$ type     : chr [1:11] "(diff)" "(diff)" "(diff)" "(diff)" ...
    ##   .. ..$ L1       : num [1:11] 0 0.0222 0 0 0 ...
    ##   .. ..$ min      : num [1:11] 0 0 0 0 0 0 0 0 0 0 ...
    ##   .. ..$ 25%      : num [1:11] 0 0 0 0 0 0 0 0 0 0 ...
    ##   .. ..$ 50%      : num [1:11] 1 0 0 0 0 0 0 0 0 0 ...
    ##   .. ..$ 75%      : num [1:11] 0 0 0 0 0 ...
    ##   .. ..$ max      : num [1:11] 0 0 0 0 0 ...
    ##   ..$ L1 :List of 4
    ##   .. ..$ L1      : num 0.605
    ##   .. ..$ breaks  :List of 10
    ##   .. .. ..$ age      : int [1:21] 16 18 20 22 24 26 28 30 32 34 ...
    ##   .. .. ..$ education: num [1:25] 3 3.5 4 4.5 5 5.5 6 6.5 7 7.5 ...
    ##   .. .. ..$ black    : num [1:6] 0 0.2 0.4 0.6 0.8 1
    ##   .. .. ..$ married  : num [1:6] 0 0.2 0.4 0.6 0.8 1
    ##   .. .. ..$ nodegree : num [1:6] 0 0.2 0.4 0.6 0.8 1
    ##   .. .. ..$ re74     : num [1:21] 0 2000 4000 6000 8000 10000 12000 14000 16000 18000 ...
    ##   .. .. ..$ re75     : num [1:20] 0 2000 4000 6000 8000 10000 12000 14000 16000 18000 ...
    ##   .. .. ..$ hispanic : num [1:11] 0 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 ...
    ##   .. .. ..$ u74      : num [1:6] 0 0.2 0.4 0.6 0.8 1
    ##   .. .. ..$ u75      : num [1:6] 0 0.2 0.4 0.6 0.8 1
    ##   .. ..$ LCS     : num 25.8
    ##   .. ..$ grouping: NULL
    ##   .. ..- attr(*, "class")= chr "L1.meas"
    ##   ..- attr(*, "class")= chr "imbalance"
    ##  - attr(*, "class")= chr "cem.match"


### 5.4 自行设定粗化的cutpoint

对于连续型变量或者类别较多的分类变量，可以通过 *cem()函数*
中cutpoints和grouping两个参数来设定粗化的分割点，以下以cutpoints为例;

从上述匹配后的结果中可以看出age变量的自动cutpoint为 17, 20.8, 24.6,
28.4, 32.2, 36, 39.8, 43.6, 47.4, 51.2, 55

<font color="#836FFF"> 如果是采用算法自动设定cutpoint，
    可以通过 *cem()函数* 中L1.breaks = "fd"等（见3. 算法）来选择不同的方法。</font>  


-   如果自行设定，需要通过cutpoints = list(education = c(0, 6.5, 8.5,
    12.5, 17))来设定，即通过list形式为需要的匹配变量 赋值一个数值向量。

-   grouping参数赋值同理，list(c("strongly agree", "agree"),
    c("neutral","no opinion"),
    c("strongly))，不过我个人比较习惯提前使用factor() 设定好分类。

### 5.5 权重weights的应用

由于CEM为不对称匹配，当一个Treat样本匹配多个Control样本时，需要通过权重来更准确的估计平均处理效应(ATT)，Gary
King的原话如下：

> They enable us to use a calculation trick that makes it easy to
> estimate the ATT in a weighted least squares regression program
> without the involved procedure.

**关于权重需要注意的三点：**

-   当匹配为不对称时，对匹配后的样本进行的所有的统计分析，都应对权重进行加权，在PSM中进行一对多匹配时同理
-   CEM中权重的理解十分简单，未匹配上的个案权重全为0，
    匹配上的Treat组个案权重都为1，
    匹配上的Control组个案的权重是对粗化后各层内
    Treat和Control组的样本比与全部样本中Treat和Control组的样本比相乘而来((m\_C/m\_T)\*Ws)
-   匹配后样本的权重之和就等于匹配后样本量的大小，如本例中sum of weigths
    = sample of matched = 179

关于权重的具体计算方法，详见[An Explanation for CEM
Weights](https://docs.google.com/document/d/1xQwyLt_6EXdNpA685LjmhjO20y5pZDZYwe2qeNoI5dE/edit) （需要科学上网）  

### 5.6 k2k进行1:1匹配

虽然CEM的优势在于可以进行非对称匹配，从而保留更多的样本，但是当样本量比较充足时，为了保证更准确的估计ATT，可以
进行1:1匹配，**cem() 包** 也给出了对应的函数 *k2k()* ，示列如下：

    # 用单独的k2k函数时，之前生成cem对象mat时，必须加上keep.all = TRUE参数
    # mat2 <- k2k(obj = mat, data = df, method = "euclidean", mpower = 1)

    # 或

    mat2 <- cem(treatment = "treated", data = df_1, drop = "re78",
                 eval.imbalance = TRUE, k2k = TRUE, method = "euclidean", mpower = 1)


进行1:1匹配时，实际采用最近距离法在各层内选取，判断距离的方法可选（‘euclidean’,
‘maximum’, ‘manhattan’, ‘canberra’, ‘binary’ and ‘minkowski’)
，默认为NULL，即随机选取。

<font color="#FF0000">
需要注意的一点是，使用k2k进行1:1匹配后，后续统计分析时就无需进行权重加权了。</font>

## 6. 参考文献
-----------

-   Stuart, Elizabeth A. (2010): “Matching Methods for Causal Inference:
    A Review and a Look Forward”. In: Statistical Science, no. 1, vol.
    25, pp. 1–21.
-   Iacus SM, King G, Porro G (2008). “Matching for Causal Inference
    Without Balance Checking.” Submitted, URL
    <http://gking.harvard.edu/files/abs/cem-abs.shtml>.
-   Stefano M Iacus, Gary King, and Giuseppe Porro. 2009. “CEM: Software
    for Coarsened Exact Matching.” Journal of Statistical Software, 30.
-   Matthew Blackwell, Stefano Iacus, Gary King, and Giuseppe
    Porro. 2009. “CEM: Coarsened Exact Matching in Stata.” The Stata
    Journal, 9, Pp. 524–546.
-   Stefano M Iacus, Gary King, and Giuseppe Porro. 2011. “Multivariate
    Matching Methods That are Monotonic Imbalance Bounding.” Journal of
    the American Statistical Association, 106, 493, Pp. 345-361.
-   Stefano M. Iacus, Gary King, and Giuseppe Porro. 2012. “Causal
    Inference Without Balance Checking: Coarsened Exact Matching.”
    Political Analysis, 20, 1, Pp. 1--24. Website Copy at
    <http://j.mp/2nRpUHQ>
-   Rosenbaum, Paul R. and Donald B. Rubin (1983): “The Central Role of
    the Propensity Score in Observational Studies for Causal Effects”.
    In: Biometrika, vol. 70, pp. 41–55.
-   Cochran, W. G. (1968), “The Effectiveness of Adjustment by
    Subclassification in Removing Bias in Observational Studies,”
    Biometrics, 24, 295–313 \[350\].
-   Why propensity scores should be used for matching," German Stata
    Users' Group Meetings 2017 01, Stata Users Group.

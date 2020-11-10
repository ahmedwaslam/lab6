---
title: "lab 6"
author: "dawa lama"
date: "11/2/2020"
output:
  word_document: default
  html_document: default
  pdf_document: default
---
```{r}
load("C:/Users/singh/OneDrive/Desktop/project new/lab 6/acs2017_ny_data.RData")
attach(acs2017_ny)
```
In this lab-6.As mention in question We are using logit and probit models. These are suited for when the dependent y variable takes values of just 0 or 1. We will look at labor force participation, which is an important aspect of the economy – who chooses to work (where those who are unemployed but looking for work are considered part of the labor force).

I am interested in examine the labor patterns and looking for job within the female population.It will be focused on that might affect women’s participation in the labor force that generally would not affect men, I will check is there lies the biasness or not.  Does number of children(MCHILD), marital status(MARST)and other miscellaneous factors to female effect on for force participation of female population. I set me minimum age of any natural person is 18 and legally they can be entered in laborforce when they turn 18 years so, we’ll restrict our data set so to include only females aged 18+.So, the age group 0-25 will cover the my both group of age 0-17 and 18-25 as 0-25.In my data set I run  This population of females aged 18+ has a total number of observations equal to 82755 with 109 variables.



```{r}
use_varb <- (AGE >= 18) & (female == 1) 
dat_use <- subset(acs2017_ny,use_varb) 
detach()
attach(dat_use)
```

Let create the data to use:

````{r }
dat_use$LABFORCE <- as.factor(dat_use$LABFORCE)
levels(dat_use$LABFORCE) <- c("Not in LF","in LF", "N/A")

dat_use$MARST <- as.factor(dat_use$MARST)
levels(dat_use$MARST) <- c("married spouse present","married spouse absent","separated","divorced","widowed","never married")
```

What is the difference between “NA” as label and Not in the Labor Force? Make sure you understand. (Hint, look at ages in each group).

In general it is a good idea to check summary stats before doing fancier models. What fraction of people, say, 55-65, are in the labor force? What about other age ranges? What would you guess are other important predictors? For example,
Ans: As a calculated below, yes age 55-65 

```{r}
library(kableExtra)
```

```{r}
dat_use$age_bands <- cut(dat_use$AGE,breaks=c(0,25,35,45,55,65,100)) #This piece of code allows to divide the variable into continues data form.
lf <- table(dat_use$age_bands,dat_use$LABFORCE) #Creates a variable containing a table of participation in the labor force by age group.

#The below piece of code creates a nice-looking table out of lf variable we created:
lf %>% 
  kbl() %>%
  kable_styling()
```
```{r}
#Here is a graph for visually showing participation in the labor force by age group:
plot(lf, main="Labor Force Participation by Age Group",
xlab="Age Group",
ylab="Participation", 
col = c("red", "blue"))
```

Since in USA people start the work after 18 because legally they are eligible to work but exactly I am not sure under 25 years female looking for job or not so I have include the female under 25(age 18-25 are working in USA) in my analysis and I found 6193 females is in Labor force in my data. In my the table above we see that the female population between 25-35 has a higher participation number in Laborforce and  above 65 has a lower participation rate than other age groups. we can see on table below
Age_group	   Not_in_LF	in LF   	N/A
(0,25]	       3778	      6193	  0
(25,35]	       2456      	10070	  0
(35,45]	       2699	      9168   	0
(45,55]	       3245	      10739	  0
(55,65]      	 6047	      8980	  0
(65,100]    	16751	      2629	  0



Now let use a Baseline model to creat the logistic regression
```{r}
model_logit1 <- glm(LABFORCE ~ AGE + MARST + NCHILD,
            family = binomial, data = dat_use)
summary(model_logit1)
library(stargazer)
stargazer(model_logit1, type="text") #Let's make it prettier and easier to interpret. 

```
In this my regression analysis we can see that all the valriables are statistically significance but  'separated' marital status is statistically insignificance. Yes there is relationship with labor force status to Marital status and no of child have by female population whose age is between 25-55.


```{r}

dat_use$LABFORCE <- droplevels(dat_use$LABFORCE)

NNobs <- length(dat_use$LABFORCE)
set.seed(12345) # just so you can replicate and get same "random" choices
graph_obs <- (runif(NNobs) < 0.1) # so something like just 1/10 as many obs
dat_graph <-subset(dat_use,graph_obs)  

 plot(LABFORCE ~ jitter(AGE, factor = 2), pch = 16, ylim = c(0,1), data = dat_graph, main = "Labor Force Participation by Age", xlab = "Age", ylab = "Labor Force Status", col = c("green","red"))


to_be_predicted <- data.frame(AGE = 25:55, MARST = "never married", NCHILD = 1)
to_be_predicted$yhat <- predict(model_logit1, newdata = to_be_predicted)

lines(yhat ~ AGE, data = to_be_predicted)
```
IN this graph we can see age group start from 15 actually that include the 18-25 year female population in labor force. In this graph we can see that some female who is over 85years old and still in labor force and working.


```{r}
#Transform "unmarried" variable into a factor:
dat_use$unmarried <- as.factor(dat_use$unmarried)
levels(dat_use$unmarried) <- c("Married","Unmarried")


model_logit2 <- glm(LABFORCE ~ AGE + unmarried + NCHILD,
            family = binomial, data = dat_use)

stargazer(model_logit2, type="text") #Let's make it prettier and easier to interpret
```
In above analysis we can see that all variables are statistically significance with '***' with 99% confidence interval even though there P value of NCHILD is higher than other its means females who have child is more likely less in laborforce. In more brief we can say that there is female with child is almost zero participation in labor force.

In model_logit3 I am going to do regression of Age, male, AFAm, Asian, hispanic, educ_HS, educ_college, marrital status, mortgage, NCHILD  is  a regression with age and age-squared with other variables

```{r}
model_logit3 <- glm(LABFORCE ~ AGE+ I(AGE^2)+female,
            family = binomial, data = dat_use)
summary(model_logit3)
```
```{r}
stargazer(model_logit3, type="text") #Let's make it prettier and easier to interpret
```
Here, we can see all variables are significance, the P value of age is more than Age square so Age is have lower participate in workforce than work square.

Now I am going to check that  sinlge white woman with college education and child in the Laborforce

```{r}
model_logit4 <- glm(LABFORCE ~ AGE + I(AGE^2) + female + AfAm + Asian  + Hispanic 
            + educ_hs + educ_somecoll + educ_college + educ_advdeg 
            + MARST + MORTGAGE+ NCHILD,
            family = binomial, data = dat_use)
summary(model_logit3)
to_be_predicted1 <- data.frame(AGE = 18:65, female = 1,white = 1, Hispanic = 0, AfAm =0, educ_hs = 0, educ_college = 1, educ_advdeg = 0 , NCHILD=1, MARST = "separated","divorced","widowed","never married" , MORTGAGE = "Yes, mortgaged/ deed of trust or similar debt" , Asian = 0 , educ_somecoll = 0)
```
In overall regression analysis we found that single female(separated from husband or single mother) with child is less participate and almost zero participate in laborforce.In above analysis we can see that all variables are statistically significance with '***' with 99% confidence interval even though there P value of NCHILD is higher than other its means females who have child is more likely less in laborforce. In more brief we can say that there is female with child is almost zero participation in labor force.

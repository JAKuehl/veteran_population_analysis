# VA Enrollment  Analysis
---

## Executive Summary
---
Every year roughly 200,000 service members leave the military, according to the Veterans Administration (VA). The VA is one of the largest components of the Federal Government outside of the Armed Forces and provides extensive services for Veterans that extends well beyond healthcare.  However, a [Congressional Research Service (CSR) report compiled in 2014](https://fas.org/sgp/crs/misc/R43579.pdf) showed only 42% of eligible Veterans enrolled in the VA.  As a result, the VA neither tracks these Veterans nor can the VA refer these veterans to other organizations or groups that provide a community.  This project attempts to identify the characteristics that indicate these individuals will not enroll and the attributes of non-enrollees.

Roughly 25% of enrolled Veterans have a service-connected disability, according to [VA Enrollment Reports](https://www.va.gov/HEALTHPOLICYPLANNING/SOE2018/2018EnrolleeDataFindingsReport_9January2019Final508Compliant.pdf).  By grouping features into themes and utilizing logistic regression, this project builds several categorical models that help identify the attributes of Veterans that do and do not enroll in the VA.

![](images/enrollment.png)
## Data Science Questions
---
1. Who are the veterans that do not enroll in the VA?  
2. Do Veterans that choose not to enroll have identifiable characteristics?  

## Data
---
For this project, I utilized the Public Use Microset data for 2018 published by the United States Census Bureau.  It represents a sample of roughly 1% of the US population.  The data is published annually with American ACS updating the data from the decennial Census.  More information about this data set and [technical documentation](https://www.census.gov/programs-surveys/acs/technical-documentation/pums/documentation.html) can be found on the Census website. 

The links below open the census FTP site to download the raw dataset.  Data is available in state-specific sets for both housing and population.  For this project, I focused on 1yr and 5 yr US population datasets.  While housing information is of considerable interest when discussing veterans, it is outside the scope fo this project.

* [2018 1 Year PUMs Dataset Download (589 MBs Zipped).](https://www2.census.gov/programs-surveys/acs/data/pums/2018/1-Year/unix_pus.zip)
* [2014-2018 5 Year PUMs Dataset Download (2.3 GBs Zipped)](https://www2.census.gov/programs-surveys/acs/data/pums/2018/5-Year/unix_pus.zip)

## Data Compression and Cleaning 
---
**NOTE** - These datasets 2.8 million records and 14.9 million records, respectively.  Processing the 5yr PUMs dataset requires a minimum of 16 GB of RAM.

* **Compression:** PUMs data is startlingly exhaustive and came in multiple sets. The 1-year PUMs arrives in 2 CSV files, and the 5-year set is in 4 CSVs.  The data dictionary is 132 pages.  However, most of the features that the Census Bureau capture are not necessarily of interest.  Removing those columns simplified the datasets, it was necessary to make the set small enough to run locally. Additionally, I removed anyone under the age of 18 from the datasets because you cannot serve in the military before that age. Only after removing this unnecessary data are the sets small enough to be combined and saved locally.  

* **Cleaning:** Only after compressing the datasets to a workable size did cleaning the data begin.  PUMs data presented some exciting challenges. First and foremost, nulls are rarely true nulls.  In many cases, a null return implies a value.  For instance, a null in ```MAR``` means never married.  Therefore, null treatment was decided on a feature by feature basis.  

## Modeling
---
Since ```HINS6``` measures VA Enrollment with a simple yes(1) or no(2), it is perfect dependent variable for classification.  I tested an array of classification models for optimal performance based on score, fit, and variance.    

**Models Tested**
1. Logisitic Regression
2. K Nearest Neighbors
3. Random Forest
4. Gaussian Naive Bayes
5. Bernoulli Naive Bayes
6. Multinomial Naive Bayes
7. Decision Trees
8. Random Forest
9. Extra Trees
10. ADA Boost
11. Guassian Process
12. Support Vector Classifier
13. Linear Support Vector Classifier

In the end, Logistic Regression consistently outperformed the other models. It demonstrated to fit better, minimize variance, take less computing power, and deliver interpretable results.  Therefore, the preferred classifier going forward is Logistic Regression. Additionally, I used both Sci-Kit Learn's ```LogisticRegression()``` and Stats Model ```Logit()``` to optimize the models.

I categorized features into groups based on similarity.  These groups helped determine if broader themes displayed predictive ability beyond the individual features.  It also helped identify multicollinearity.

Due to the high number of features available, I did try feature reduction or embedding using Principal Component Analysis (PCA).   I identified these features as "Z_train" and "Z_test" sets.  Unfortunately, PCA did not improve scores and increased the models' variance, so I did not use it for the final models.

## Findings
---

The baseline model for the VA Enrollment feature, ```HINS6```, is .697.   

![](images/model_tree.png)
   
* **Veteran Feature Group** - This is the best-performing feature group with a score of .81.  Considering the highly subjective nature of the data, any scores above .8 demonstrate a very good fit.  Additionally, the test score of .81 indicates minimal variance.  
* **Health Group** -  Similarly, the Health group with a train score of .772 and a test score of .772, which demonstrates a good fit with almost no variance.  However, most of the statistically significant features only had a probability between .55 and .45, except for Disability Rating   (```DRAT```).  Probabilities like that are little better than a coin toss.
* **Work/Income Group** - The Income Group had a train and test of a score of .716.  This is only a very modest improvement over the Baseline. Like the previous model, only one feature, Other Income adjusted for inflation(```OIP_adj```), had a significant probability.
* **Ethnicity Group** - While this group contained several features that proved to be statistically significant, the model did not improve over the Baseline.
* **Lifestyle Group** - This model did not improve over the Baseline, nor did any feature prove statistically significant.
* **General Group** - While this group contained several features that proved to be statistically significant, the model as a whole did not improve over the Baseline.
* **Education Group** - While this group contained several features that proved to be statistically significant, the model as a whole did not improve over the Baseline.
* **Location Group** - This model did not improve over the Baseline, nor did any feature prove to be statistically significant.


#### Final Combined Model
For the final model, I tested every feature that demonstrated statistical significance in the previous groups.  Then dismissed any feature that did not achieve statistical significance in the larger model.  

The final model used 41 features, produced a score of .793 for both the test and train data, and improved .1 on the Baseline.  Like the Group Models, most of the statistically significant features have indecisive probabilities between .45 and .55. Several features did stand out with more substantial probabilities.  To be clear, the closer the probability is to 1, the LESS likely the veteran will enroll in the VA.

* Has Disability Rating (```DRATX```)   - .643
* Military Service (```MIL```)     - .256
* Did not Serve between 1975-1990 (```MLPCD_0```) - .097
* Has TRI-CARE Health Insurance (```FHINS5P```) - .321
* Attained Bachelor's Degree (```SCHL_21```) - .064
* Attained Master's Degree (```SCHL_22```) - .125
* Attianed Professional Degree beyond Master's (```SCHL_23```) - .257
* Attained Doctorate (```SCHL_24```) - .312
* Science or Engineering Degree (```SCIENGP```) - .963



## Project Organization
---
```
project-directory  
|__ 1yr_pums  
|   |__ Place 1yr dataset CSVs in this directory
|__ 5yr_pums 
|__ |__ Place 5yr dataset CSVs in this directory
|__ background 
|__ |__ Veteran data for comparison and future research
|__ code
|   |__ 01_data_import_and_combine.ipynb
|   |__ 02_data_cleaning.ipynb   
|   |__ 03_classification.ipynb  
|__tableau_data
|   |__ 01_tableau_build.ipnyb
|   |__ pop1_analysis.twb
|   |__ vet1_analysis.twb
|   |__ Tableau CSVs will be saved here
|__ visuals
|__ work_in_progress
|__ |__ Clustering.ipnyb
|__ |__ EDA_visuals.ipnyb
|__ working_data
|   |__ Compressed and cleaned CSVs will be saved here
|__ presentation.pdf
|__ README.md
```

## Follow On Work
---
This data set contains a great amount of incites that are outside of today's project.  Several ways I plan to continue work on this data set:

* Attempt classification using Neural Networks 
* Perform a deeper comparison veteran traits versus the general population
* Predict income variables based on enrollment and other veteran features
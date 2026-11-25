# Customer churn prediction on Online Retail II



## The data

I used the Online Retail II dataset from the UCI repository: https://archive.ics.uci.edu/dataset/502/online+retail+ii (DOI 10.24432/C5CG6D, licence CC BY 4.0). It has about 1.07 million invoice lines from a UK online gift shop, from December 2009 to December 2011.

There is no "churned" column in it, so I had to decide what churn means myself. That turned out to be the most important part of the project.

The data file is not in this repo. Download it and save it as `online_retail.xlsx` next to the notebook. The two sheets should be called `Year 2009-2010` and `Year 2010-2011`.

## How I defined churn

I picked a cutoff date, 1 July 2011.

- Everything before the cutoff is used to describe each customer (the features).
- Everything after it, up to 9 December 2011 (162 days), is only used to check whether the customer came back (the label).
- A customer who bought before the cutoff and did not buy again in those 162 days counts as churned.

Features and label never use the same period, so the model cannot see the future. I also checked the window length against normal buying habits: only 6.6% of the gaps between purchases before the cutoff were longer than 162 days.

## What the notebook does

1. **Cleaning.** I dropped rows with no customer ID, exact duplicates, invoices that start with C (cancellations, kept aside to count them later), non-product codes like POST, and rows with zero or negative quantity or price. I also removed purchases that were fully cancelled later, found by matching customer, product and quantity. 1,067,371 rows became 770,398.
2. **A bit of EDA** to understand the data.
3. **Features.** Six per customer, from before the cutoff only: recency, frequency, total spend, average order value, tenure and number of cancellations.
4. **Split.** 80/20 by customer: 4,006 for training and 1,002 for testing.
5. **Models.** A dummy baseline, logistic regression and random forest, compared with 5-fold cross-validation on the training set only. The random forest was tuned with a random search. Before looking at results I set a rule: keep the forest only if it beats logistic regression by at least 0.005 ROC-AUC.
6. **Threshold.** Chosen from cross-validated training predictions: the highest threshold that still catches 90% of churners.
7. **Test.** The test set is used once at the end. After that I looked at a decile table, feature importance and the churners the model missed.

## Results

All of these come from the notebook run.

- 5,008 customers were modelled and 49.7% of them churned.
- Cross-validated ROC-AUC: logistic regression 0.802, random forest 0.796, tuned random forest 0.807. The tuned forest was chosen.
- Test ROC-AUC is 0.814 (the dummy baseline gets 0.500).
- At threshold 0.35 the model catches 90.0% of churners (448 of 498) with 64.8% precision. Accuracy is 70.8% there (73.6% at the usual 0.5, and 50.3% for the baseline).
- Recency is by far the most important feature.
- The 10% of test customers with the highest predicted risk churned 91.1% of the time, and the top 20% contain 34.5% of all churners.
- 32.3% of customers bought on only one day. 74.7% of them churned, against 37.8% of repeat buyers.

The gap between the two models is small (0.807 against 0.802). The forest passes my 0.005 rule, but only just, so logistic regression would also have been a fair choice.

## Things to keep in mind

- I only tried one cutoff date and did not test on a later time period.
- The label window includes the Christmas season, which may mislabel seasonal shoppers.
- "No purchase in 162 days" is a stand-in for leaving. Some of these customers may come back later.
- The 0.35 threshold flags about 70% of customers. That only makes sense if contacting a customer is cheap compared with losing one.
- Matching cancellations is a rule of thumb and may occasionally remove a real purchase.
- There is only purchase history here: no marketing data, product categories or customer details.

## Running it

```
pip install -r requirements.txt
jupyter notebook customer_churn.ipynb
```

Run all cells from the top. Loading the Excel file takes a while the first time.

I ran it on Windows 11 with 15 GB RAM, Python 3.13.14, pandas 3.0.3, numpy 2.4.6 and scikit-learn 1.9.0.

## Files

- `customer_churn.ipynb`: the whole project
- `requirements.txt`: Python packages

## Credits

Dataset: Chen, D. (2012). Online Retail II [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5CG6D



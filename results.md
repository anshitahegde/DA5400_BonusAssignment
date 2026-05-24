So a couple of deviations in implementation which don't lead to full-scale reproduction of results
1. The swapping of the MIO algorithm for the CART algorithm, which is not strong against noisy data. So the trees are not optimised to begin with
2. The specific data set chosen and the binarisation of data can lead to some loss of information

On comparison with the 3 methods 
1. random foresting 2. SVM 3. Logistic regression
the robust tree method is not a clear winner in many cases, but there are key factors to take into account.
Random forests are aggregates of trees, making it uninterpretable. SVM, which may have strong worst-case accuracy, is not that good in average accuracy.
Further in the paper, the comparison is done between a robust and a non-robust tree.

 

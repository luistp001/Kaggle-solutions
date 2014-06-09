Methodology:
============
The final prediction is a weighted average of my code (2/3) and Ben's (1/3).
 
Benjamin Solecki
================
Ben's approach is itself a blend of a logistic model and a mixture of tree-based models. He explained his approach for the logistic model in more detail here. As for the tree-based models, it is a combination of Random Forests, GBMs, and Extremely Randomized Trees that are grown using features based on counts and frequencies (e.g. number of resources managed by a given manager, etc.). I'll let him explain his approach further if he wishes to -- we merged comparatively late in the game (2 weeks before the deadline) so I would risk misrepresenting his train of thoughts.
 
Paul Duan
=========
As for mine, it was mainly driven by the fact that the structure of the dataset contained all categorical variables, with a large number of categories and some rare features. This meant that stability was a high priority, ie. models should be relatively robust to changes in the composition of the dataset. I believe this is what in the end helped our score to drop less than the other top competitors from the public leaderboard to the private one.
As such:
- I trusted my CV score above all else, which was obtained by repeating a 80/20 train/test split 10 times. I would then select the models not only based on raw score, but also on the standard deviation between the 10 folds.
I made no attempt at fixing the discrepancy between CV and leaderboard scores that was due to the fact that all categories in the test set appeared in the train set, which was not necessarily the case when doing a CV split. The reasoning being that the best model would need to be resistant to a change in the split.
- I removed role 1 and role 2 from the original columns, as their effect was too strong and seemed to cause too much variance; I suspect that the algorithms weren't too good at dealing with unknown role IDs (they were the ones with the fewest number of categories, so the models tended to trust them too much)
- I spent comparatively very little time on feature selection (which was a big subject in this competition, judging by the forums), as they would be highly dependent on the makeup of the dataset. I did, however, reuse some of Miroslaw's code to generate three different datasets built by using greedy feature selection with different seeds. This was not critical to the raw performance of the blend but did help diversifying it/reducing the variance.
- I considered feature extraction to be much more important to feature selection
- my classifier consists of a large combination of models (~15 currently) that are each either using a different algorithm or a different feature set. The top 3-5 models are probably enough to reach .92+ on the private leaderboard, but I found adding additional variants to the datasets helped minimize variance.
I then combined them by computing their predictions using cross-validation, and combining them using a second model (stacking). When training the second model, I also added meta-features (nb of time this resource/manager/etc appeared in the training set, etc.), the idea being to try to determine dynamically which model should be trusted the most (some perform better when the manager is unknown, etc.).
 Each dataset consists of what I called base data sets (combination of the original columns) and extracted feature sets.
The extracted feature sets are based on cross-tabulating all categories and looking at the counts manager/resource/etc X had been associated with role/department/etc Y, and so on (in feature_extraction.py, this is what I called the pivot tables; the features sets are lists of lambda functions that extract the relevant cell in the table).
 
Usage
=====
To run the code, you'll need Python, numpy/scipy and pandas installed. Pandas is not really necessary for the most part, but it is used in some of the external code; I may rewrite them to remove this dependency when I have the time.
The usage is described on the Github page (to create a submission made up of a simple average of the models in my blend, simply run python classifier.py -v -f testsubmission and it should show up in your submissions/ folder. This should net you a private leaderboard score of .92330. To get the full .92360, you'll need to also run BSMan's code in the BSMan/ folder then combine all predictions according to a 2/3 1/3 split, where BSMan's final prediction is itself a 2/3 1/3 combination of his logistic model and ensemble models.
Runtime for my models should be ~30 minutes per fold, considerably less if you remove some models (simply comment out some strings in the model list, especially the GBC ones), and considerably more if you enable stacking as this will require cross-validated predictions on the training set to be computed as well.
Everything is cached so you'll never have to compute the same thing twice -- it also means that you should have some disk space available (the cache may grow to a couple gigabytes).

# Anil's Solution for OTTO – Multi-Objective Recommender System

The solution pipeline consists of:
- [Generating Covisitation Matrices](https://github.com/nlztrk/OTTO-Multi-Objective-Recommender-System/blob/main/0.%20Covisitation.ipynb)
- [Splitting the Data as Train/Val](https://github.com/nlztrk/OTTO-Multi-Objective-Recommender-System/blob/main/1.%20Generating%20Splits.ipynb)
- [Generation Co-Occurrence Matrices](https://github.com/nlztrk/OTTO-Multi-Objective-Recommender-System/blob/main/2.5%20Generating%20Co-Occurrences.ipynb)
- [Candidate Generation](https://github.com/nlztrk/OTTO-Multi-Objective-Recommender-System/blob/main/2.%20Candidate%20Generation.ipynb)
- [Feature Extraction](https://github.com/nlztrk/OTTO-Multi-Objective-Recommender-System/blob/main/3.%20Feature%20Extraction.ipynb)
- [Training](https://github.com/nlztrk/OTTO-Multi-Objective-Recommender-System/blob/main/4.%20Training.ipynb)
- [Inference](https://github.com/nlztrk/OTTO-Multi-Objective-Recommender-System/blob/main/5.%20Inference.ipynb)

## Generating Covisitation Matrices
Generated **Top-100** AIDs for three well-known covisitation schemes given in: [link](https://www.kaggle.com/code/tuongkhang/otto-pipeline2-lb-0-576)

## Splitting the Data as Train/Val
Used the [local validation scheme](https://www.kaggle.com/datasets/radek1/otto-train-and-test-data-for-local-validation) given by [Radek](https://www.kaggle.com/radek1). The local train data was also splitted into two by sessions in order to avoid possible leakage during model training and score calculation. The implementation can be seen in the corresponding notebook.

## Generating Co-Occurrence Matrices
Generated all pair occurrences for all AIDs among all sessions for all action pairs (click-cart, cart-order, etc.). This is used for feature extraction later.

## Candidate Generation
Used the [public candidate generation script](https://www.kaggle.com/code/tuongkhang/otto-pipeline2-lb-0-576) and generated **100** candidates for all action types.

## Feature Extraction

Generated features for following data subsets:
- Items
- Sessions
- Item-Session Combinations
- Covisitation and Co-Occurrence Statistics

### Item Features
- Statistics generated from hour, weekday and weekend status
- Count features (bool for >0 and >1, rank among all)
- Unique count features (unique count and rank among all)
- Distribution of action types in percentiles
- Inclusion rate by all sessions
- Occurrence rate in the last week of data
- Average number of times seen in the same sessions at different times
- All of the above with filtered separately for all action types

### Session Features
- Statistics generated from hour, weekday and weekend status
- Count features (bool for >0 and >1, rank among all)
- Unique count features (unique count and rank among all)
- Distribution of action types in percentiles
- Length of the session
- Features generated by extracting mini-sessions according to the time differences between actions
- Statistics generated from multiple purchases made in a single basket
- Rates of taking products to the next action within the same session (click->cart, cart->order)
- All of the above with filtered separately for all action types

### Item-Session Combination Features
- Statistics generated from hour, weekday and weekend status
- Count features (bool for >0 and >1, rank among all)
- Unique count features (unique count and rank among all)
- Distribution of action types in percentiles
- Reversed order of the item in the session
- Time difference between the latest occurrence of the item and the start - end of the session

### Covisitation and Co-Occurrence Statistics
- Statistics generated from covisitation and co-occurrence scores between candidate items and items in the session's history

## Training

Used the following config:
- **Model:** XGBoost
- **Fold Scheme:** 5-Fold (Grouped by "session")
- **Negative Sampling Fraction:** 15%
- Dropped sessions with no positive labels
- Used the first half of splitted local training set

## Inference

- Used mean blending
- Executed on the second half of splitted local training set when running local validation

## Didn't Work & Improve

- Weekday-Specific aggregations
- Word2Vec features
- Different models (CatBoost, LGBM)
- Comprehensive pair scores (because of OOM errors)
- Max-median blending
- Early-stopping
- Higher negative fractions
- Different objective metrics
- Different fold counts

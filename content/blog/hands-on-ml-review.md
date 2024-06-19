+++
title = 'Hands on Ml Review'
date = 2024-06-08T21:17:57+05:30
draft = true
tags = ["Machine learning"]
+++

Hands on Machine Learning is a book which is recommended by many to be used as a first ML book to learn how to code ML models. I started reading this book back in January but left it in between. But thought of giving this book a serious read this time and finish this. Today is June 8 and I'm starting to journal this journey and will eventually post this as a blog post(which you are reading now :)

## Chapter 1-2

The book starts slow with the introduction to the ML landscape in the first chapter. It goes through the types of machine learning algorithms and the main challenges faced while building ML models.

The meaty part of the book starts with the second chapter. This chapter walks you through an end-to-end machine learning project, emphasizing on the basics and how the overall process looks like in the real world.

One thing which I liked in this chapter was first the author demonstrated how to do one task using just numpy and python and then did the same thing in one line using scikit-learn.

I really liked this elegant method of implementing train_test_split:
```python
from zlib import crc32

def test_set_check(identifier, test_ratio):
	return crc32(np.int64(identifier)) & 0xffffffff < test_ratio * 2**32

def split_train_test_by_id(data, test_ratio, id_column):
	ids = data[id_column]
	in_test_set = ids.apply(lambda id_: test_set_check(id_, test_ratio))
	
	return data.loc[~in_test_set], data.loc[in_test_set]

```

It take the California housing price prediction dataset. Although this is a regression task, this books takes it up a notch with introducing concepts like Stratified sampling, Cross validation splits, model tuning, etc.

Main emphasis is given on the Data cleaning process. Care is taken that all possible edge cases which you may find while building a model yourself are covered in this chapter.

The following concepts were taught in this chapter:
- Stratified shuffling
- Correlation matrix
- Imputing data
- Ordinal and One Hot Encoding
- How to make custom transformers using scikit-learn.


## Chapter 3

This chapter is about classification problems and how to deal with them. Takes the good'ol MNIST dataset to teach classification algorithms.

Majority of the time is spend on making sure you understand the performance measure stuff like Recall, Precision, ROC, F1 Score, 

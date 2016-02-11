
# Feature Flocks: Accurate Pattern Discovery in Multivariate Signals

## What problem does this solve?

### Short version

We take in a multivariate time series containing some unknown pattern and return both a model of the pattern and all the places it happens.

### Long version

Thanks to the rise of wearable and connected devices, sensor-generated time series comprise a large and growing fraction of the world's data. Unfortunately, extracting value from this data can be challenging, since sensors can only report low-level signals (e.g., acceleration), not the high-level phenomena that are typically of interest (e.g., gestures).

Spotting such high-level phenomena using low-level signals is currently problematic. Given enough labeled examples of the phenomena taking place, one could, in principle, train a classifer for this purpose. However, obtaining labeled examples is extremely difficult. Unlike text or images, time series cannot be culled at scale from the web, and even if they could be, they are difficult or impossible for humans to inspect and label. The alternative is to record examples yourself, but as those of us who've spent time doing this know, it "require[s] huge effort from human annotators or experts."[1](http://users.ece.cmu.edu/~hengtzec/papers/MobiSys13_NuActiv_Cheng_CMU.pdf). Perhaps worst of all, both of these approaches require one to know all the phenomena that exist to be labeled, which may not be the case.

What is needed, then, is an automated means of discovering and labeling patterns within time series, and this is what our algorithm does.

## How does it do this?

The basic idea is to represent the time series in terms of what features are present over time. Example features include levels of variance, slope, or resemblence to learned [shapelets](http://alumni.cs.ucr.edu/~lexiangy/Shapelet/kdd2009shapelet.pdf). Using this representation, a pattern consists of certain features occurring together in time more often than would be expected by chance. We term this set of features a "flock" and have a fast algorithm for finding it.

## How do I use this code?

If you want to use our algorithm, just use the code in `standalone/`. It's <300 lines, excluding the many comments. There's an example of loading in a list of time series, finding the patterns, and plotting the results in `standalone/main.py`. You can run it via:
```
$ python -m <repoDirectory>/standalone/main.py
```
Further, if you're trying to find patterns in a particular domain, you can likely get better performance using customized features. The current implementation only uses shape features since these are fairly generic, but any sparse features valued in [0, 1] will work. For example, quantizing and one-hot encoding the raw time series often works well if there's no wandering baseline or amplitude variability.

If you want to use our datasets, look in `python/datasets/`. There are files to read in each dataset (as well as some others not used in the paper), and a top-level wrapper function `loadDataset()` in datasets.py. Note that you will have to download the raw datasets yourself and edit `paths.py` to point to the proper locations on your machine.

If you want to dig into the details of our experiments, look in the `python` subdirectory. Aside from the contents of `python/datasets`, this code is less well-documented, more complex, and not recommended for use.

## Dependencies

### Standalone code:
None besides the Scipy stack, although main.py uses the datasets

### Datasets:
[Librosa](https://github.com/bmcfee/librosa) - for extracting MFCCs for the TIDIGITS dataset
[Joblib](https://github.com/joblib/joblib) - for caching function output
[FFmpeg](http://www.ffmpeg.org) - optional, for animating the subsequences in a dataset
[AMPDs](http://ampds.org) - the dishwasher dataset
[TIDIGITS](https://catalog.ldc.upenn.edu/LDC93S10) - the TIDIGITS dataset
[MSRC-12](http://research.microsoft.com/en-us/um/cambridge/projects/msrc12/) - the MSRC-12 Kinect dataset
[UCR Archive](http://www.cs.ucr.edu/~eamonn/time_series_data/) - the UCR time series archive

### Full codebase (i.e., if running the experiments):
[Numba](https://github.com/numba/numba) - used to speed up comparison algorithms
[Scikit-learn](https://github.com/scikit-learn/scikit-learn) - for building pipelines
[Seaborn](https://github.com/mwaskom/seaborn) - for creating certain figures

## Does it work?

Yes. It's not as smart as a human staring at only the relevant signals, but it's *much* more accurate (and much faster) than alternative algorithms.

Experimental details are given in the paper, but here's an overview of how it performed in terms of accuracy and speed.

### Accuracy

![AccuracyResults](/figs/web/accuracy.jpg?raw=true)

Higher lines are better. This says that when you compare the places in the data that the algorithms thought were instances of the pattern with those that actually were, ours is right much more often. The x axis is how stringent a cutoff we have for being "right" (defined in terms of how much a predicted instance has to overlap in time with a true instance). The y axis is the [F1 Score](https://en.wikipedia.org/wiki/F1_score), a measure of how well it retrieves (only) the true instances.

### Scalability

![ScalabilityResults](/figs/web/scalability.jpg?raw=true)

Lower lines are better. This says that when you increase the length of the time series, the estimated length of the pattern, or the range of possible lengths of the pattern (since you don't have to know it exactly), our algorithm doesn't get much slower. Further, it's virtually always 10x-100x faster than alternatives.


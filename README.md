# UA-GEC: Grammatical Error Correction and Fluency Corpus for the Ukrainian Language

This repository contains UA-GEC data and an accompanying Python library.


## Data

All corpus data and metadata stay under the `./data`. It has two subfolders
for [train and test splits](#train-test-split)

Each split (train and test) has further subfolders for different data
representations:

`./data/{train,test}/annotated` stores documents in the [annotated format](#annotation-format)

`./data/{train,test}/source` and `./data/{train,test}/target` store the
original and the corrected versions of documents. Text files in these
directories are plain text with no annotation markup. These files were
produced from the annotated data and are, in some way, redundant. We keep them
because this format is convenient in some use cases.


## Metadata

`./data/metadata.csv` stores per-document metadata. It's a CSV file with
the following fields:

- `id` (str): document identifier.
- `author_id` (str): document author identifier.
- `is_native` (int): 1 if the author is native-speaker, 0 otherwise
- `region` (str): the author's region of birth. A special value "Інше"
  is used both for authors who were born outside Ukraine and authors
  who preferred not to specify their region.
- `gender` (str): could be "Жіноча" (female), "Чоловіча" (male), or "Інша" (other).
- `occupation` (str): one of "Технічна", "Гуманітарна", "Природнича", "Інша"
- `submission_type` (str): one of "essay", "translation", or "text\_donation"
- `source_language` (str): for submissions of the "translation" type, this field
    indicates the source language of the translated text. Possible values are
    "de", "en", "fr", "ru", and "pl".
- `annotator_id` (int): ID of the annotator who corrected the document.
- `partition` (str): one of "test" or "train"
- `is_sensitive` (int): 1 if the document contains profanity or offensive language

## Annotation format

Annotated files are text files that use the following in-text annotation format:
`{error=>edit:::error_type=Tag}`, where `error` and `edit` stand for the text item before
and after correction respectively, and `Tag` denotes an error category
(`Grammar`, `Spelling`, `Punctuation`, or `Fluency`).

Example of an annotated sentence:
```
    I {likes=>like:::error_type=Grammar} turtles.
```

An accompanying Python package, `ua_gec`, provides many tools for working with
annotated texts. See its documentation for details.


## Train-test split

We expect users of the corpus to train and tune their models on the __train__ split
only. Feel free to further split it into train-dev (or use cross-validation).

Please use the __test__ split only for reporting scores of your final model.
In particular, never optimize on the test set. Do not tune hyperparameters on
it. Do not use it for model selection in any way.

Next section lists the per-split statistics.


## Statistics

UA-GEC contains:

| Split     | Documents | Sentences |  Tokens | Authors |
|:---------:|:---------:|----------:|--------:|:-------:|
| train     | 851       | 18,225    | 285,247 | 416     |
|  test     | 160       | 2,490     | 43,432  | 76      |
| **TOTAL** | 1,011     | 20,715    | 328,779 | 492     |

See [stats.txt](./stats.txt) for detailed statistics generated by the following
command (`ua-gec` must be [installed](#getting-started) first):

```
$ make stats
```


## Python library

Alternatively to operating on data files directly, you may use a Python package
called `ua_gec`. This package includes the data and has classes to iterate over
documents, read metadata, work with annotations, etc.

### Getting started

The package can be easily installed by `pip`:

```
    $ pip install ua_gec==1.1
```

Alternatively, you can install it from the source code:

```
    $ cd python
    $ python setup.py develop
```


### Iterating through corpus

Once installed, you may get annotated documents from the Python code:

```python
    
    >>> from ua_gec import Corpus
    >>> corpus = Corpus(partition="train")
    >>> for doc in corpus:
    ...     print(doc.source)         # "I likes it."
    ...     print(doc.target)         # "I like it."
    ...     print(doc.annotated)      # <AnnotatedText("I {likes=>like} it.")
    ...     print(doc.meta.region)    # "Київська"
```

Note that the `doc.annotated` property is of type `AnnotatedText`. This
class is described in the [next section](#working-with-annotations)


### Working with annotations

`ua_gec.AnnotatedText` is a class that provides tools for processing
annotated texts. It can iterate over annotations, get annotation error
type, remove some of the annotations, and more.

While we're working on a detailed documentation, here is an example to
get you started. It will remove all Fluency annotations from a text:

```python
    >>> from ua_gec import AnnotatedText
    >>> text = AnnotatedText("I {likes=>like:::error_type=Grammar} it.")
    >>> for ann in text.iter_annotations():
    ...     print(ann.source_text)       # likes
    ...     print(ann.top_suggestion)    # like
    ...     print(ann.meta)              # {'error_type': 'Grammar'}
    ...     if ann.meta["error_type"] == "Fluency":
    ...         text.remove(ann)         # or `text.apply(ann)`
```



## Contributing

* The data collection is an ongoing activity. You can always contribute
  your Ukrainian writings or complete one of the writing tasks at
  https://ua-gec-dataset.grammarly.ai/

* Code improvements and document are welcomed. Please submit a pull request.


## Citation

The [accompanying paper](https://arxiv.org/abs/2103.16997) is:

```
@misc{syvokon2021uagec,
      title={UA-GEC: Grammatical Error Correction and Fluency Corpus for the Ukrainian Language},
      author={Oleksiy Syvokon and Olena Nahorna},
      year={2021},
      eprint={2103.16997},
      archivePrefix={arXiv},
      primaryClass={cs.CL}}
```


## Contacts

* oleksiy.syvokon@gmail.com
* olena.nahorna@grammarly.com
* nastasiya.osidach@grammarly.com

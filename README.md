![alt text](docs/logo.svg)

[![PyPI version](https://badge.fury.io/py/pycorrector.svg)](https://badge.fury.io/py/pycorrector)
[![Contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License Apache 2.0](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](https://github.com/shibing624/pycorrector/LICENSE)
![Language](https://img.shields.io/badge/Language-Python-blue.svg)
![Python3](https://img.shields.io/badge/Python-3.X-red.svg)


# pycorrector

Chinese text error correction tool. Sound-like and shape-like typos (or variants) correction, which can be used to correct errors in Chinese pinyin and stroke input methods. Python3.6 development.

**pycorrector** detects the location of typos based on the language model, and corrects the typos based on the phonetic features of pinyin, the edit distance feature of strokes and five strokes, and the perplexity feature of the language model.

## Demo

https://www.borntowin.cn/product/corrector

## Question

Chinese text error correction tasks, common error types include:

-Homophonic words, such as with a pair of eyes-with a pair of glasses
-Confusing words, such as The Wandering Weaver Girl-Cowherd and Weaver Girl
-The word order is reversed, such as Woody Allen-Allen Woody
-Word completion, such as love has providence-if love has providence
-It looks like a word error, such as sorghum-sorghum
-Full Chinese Pinyin, such as xingfu-幸福
-Chinese pinyin abbreviations, such as sz-Shenzhen
-Grammatical errors, such as unimaginable-unimaginable

Of course, for different business scenarios, these problems may not all exist. For example, input methods need to deal with the first four types, search engines need to deal with all types, and text error correction after speech recognition only needs to deal with the first two.
Among them, the "word-like error" is mainly for Wubi or stroke handwriting input.


## Solution
### Rules of solving ideas
1. Chinese error correction is divided into two steps, the first step is error detection, and the second step is error correction;
2. In the error detection part, the words are cut by the Chinese word segmenter. Because the sentence contains typos, the result of word segmentation is often incorrect. In this way, errors are detected from both word granularity and word granularity.
Integrate the suspected error results of these two granularities to form a candidate set of suspected error locations;
3. The error correction part is to traverse all the suspected wrong positions and replace the words in the wrong position with the sound and shape dictionary, and then calculate the sentence perplexity through the language model, compare and sort all the candidate set results, and get the best corrected word .

### Deep model solutions
1. The end-to-end deep model can avoid manual feature extraction and reduce manual workload. The RNN sequence model has a strong ability to fit text tasks. rnn_attention won the first place in the English text error correction competition, which proves that the application effect is good;
2. CRF will calculate the conditional probability of the global optimal output node. The detection of specific error types in the sentence will determine the error based on the entire sentence. Ali participated in the 2016 Chinese grammar error correction task and won the first place, proving that the application effect is good ；
3. The seq2seq model uses the encoder-decoder structure to solve sequence conversion problems. It is currently one of the most widely used and best-effect models in sequence conversion tasks (such as machine translation, dialogue generation, text summarization, and image description).


## Feature
### Model
* kenlm: kenlm statistical language model tool
* rnn_attention model: refer to the nlc model of Stanford University, which is a method for participating in the 2014 English text error correction competition and winning the first place
* rnn_crf model: refer to Alibaba's 2016 Chinese Grammar Error Correction Contest CGED2018 and the method of winning the first place (finishing)
* seq2seq_attention model: adding the attention mechanism to the seq2seq model has better effect on long texts, and the model is easier to converge, but it is easy to overfit
* Transformer model: The full attention structure replaces lstm to solve the sequence to sequence problem, and the semantic feature extraction effect is better
* bert model: Chinese fine-tuned model, using MASK feature to correct typos
* conv_seq2seq model: Based on the fairseq produced by Facebook, the Beijing Language and Culture University team improved the ConvS2S model for Chinese error correction. In the NLPCC-2018 Chinese grammatical error correction competition, it was the only single model and won the third place.
* Electra model: A more efficient pre-training model jointly proposed by Stanford and Google, learning text context representation is better than BERT and XLNet with the same computing resources

### Error detection
* Word granularity: The language model perplexity (ppl) detects that the likelihood of a word is lower than the average value of the sentence text, and the probability of determining that the word is a suspected typo is high.
* Word granularity: The probability that a word that is not in the dictionary after the word is cut is a suspected wrong word.


### Error correction
* After locating all suspected errors through error detection, take all the candidate words that are similar in sound and shape to the suspected typo,
* Use candidate word replacement, based on the language model to get the candidate ranking results similar to the translation model, and get the best corrected word.


### Thinking
1. The current processing methods are not bad in terms of error recall at word granularity, but the accuracy of error correction needs to be improved. More high-quality error correction sets and error correction vocabulary will be improved. I hope that the algorithm will be bigger Breakthrough.
2. In addition, the current text errors are no longer limited to spelling errors in word granularity. It is necessary to improve the Chinese Grammar Error Diagnosis (CGED, Chinese Grammar Error Diagnosis) and correction capabilities, which are listed in TODO for follow-up research.



## Install
* Fully automatic installation: pip install pycorrector
* Semi-automatic installation:
```
git clone https://github.com/shibing624/pycorrector.git
cd pycorrector
python setup.py install
```


The installation can be completed by either of the above two methods. If you don’t want to install it, you can download [github source package](https://github.com/shibing624/pycorrector/archive/master.zip), install the following dependencies before using.

#### Installation dependencies
* kenlm installation
```
pip install https://github.com/kpu/kenlm/archive/master.zip
```

* Other library package installation
```
pip install -r requirements.txt
```

## Usage

-Text error correction

```python
import pycorrector

corrected_sent, detail = pycorrector.correct('The young pioneers should sit down because they should be old people')
print(corrected_sent, detail)
```

output:
```
Young pioneers should give up their seats for the elderly [[('Because of','should', 4, 6)], [('sit','seat', 10, 11)]]
```

> The rule method will load the kenlm language model file from the path `~/.pycorrector/datasets/zh_giga.no_cna_cmn.prune01244.klm` by default. If the file is not detected, the program will automatically download it online. Of course, you can also manually download the [model file (2.8G)](https://deepspeech.bj.bcebos.com/zh_lm/zh_giga.no_cna_cmn.prune01244.klm) and place it in this location.


-Error detection
```python
import pycorrector

idx_errors = pycorrector.detect('The young pioneers should sit down for the elderly')
print(idx_errors)
```

output:
```
[['Cause this', 4, 6,'word'], ['sit', 10, 11,'char']]
```
> The return type is `list`, `[error_word, begin_pos, end_pos, error_type]`, and the index position of `pos` starts at 0.


-Turn off word granularity error correction
```python
import pycorrector

error_sentence_1 ='My throat is inflamed, I need to buy some Amoxicillin'
correct_sent = pycorrector.correct(error_sentence_1)
print(correct_sent)

```

output:
```
'My throat is inflamed, I need to buy some Amoxicillin', [['Xi Lin','Xi Lin', 12, 14], ['eat','Ji', 14, 15]]
```

In the above example, error correction occurs in `eat`, the following code turns off the word granularity error correction:
```python
import pycorrector

error_sentence_1 ='My throat is inflamed, I need to buy some Amoxicillin'
pycorrector.enable_char_error(enable=False)
correct_sent = pycorrector.correct(error_sentence_1)
print(correct_sent)

```

output:
```
'My throat is inflamed, I want to buy some amoxicillin to eat', [['Xi Lin','Xi Lin', 12, 14]]
```

Both the default word granularity and word granularity error correction are turned on. Generally, there are fewer single word errors, and the accuracy of word granularity error correction is low. Turn off the word granularity error correction, which can improve the accuracy and speed of error correction.

> The default `enable` parameter of the `enable_char_error` method is `True`, that is, typos correction is turned on. This method can recall the word granularity error, but the overall accuracy rate will be low;

> If you are pursuing accuracy and not recall, it is recommended to set `enable` to `False` and only use wrong words to correct.


-Load custom obfuscation sets

By loading a custom confusion set, users are supported to correct known errors, including two functions: 1) Error compensation and recall; 2) Manslaughter and whitening.

```python
import pycorrector

pycorrector.set_log_level('INFO')
error_sentences = [
    'How much does it cost to buy an iPhone?',
    'Joint actual controllers Xiao Hua, Huo Rongquan, Zhang Qikang',
]
for line in error_sentences:
    print(pycorrector.correct(line))

print('*' * 53)
pycorrector.set_custom_confusion_dict(path='./my_custom_confusion.txt')
for line in error_sentences:
    print(pycorrector.correct(line))

```

output:
```
('How much does it cost to buy an iPhone bad', []) # "iPhone bad" missed the call, it should be "iphoneX"
('Common actual controllers Xiao Hua, Huo Rongquan, Zhang Qikang', [['Zhang Qikang','Zhang Qikang', 14, 17]]) # "Zhang Qikang" manslaughter, should not be corrected
************************************************** ***
('How much does it cost to buy iPhoneX', [['iPhoneX','iPhoneX', 1, 8]])
('Common actual controllers Xiao Hua, Huo Rongquan, Zhang Qikang', [])
```

For the specific demo, see [example/use_custom_confusion.py](./examples/use_custom_confusion.py), where the content format of `./my_custom_confusion.txt` is as follows, separated by spaces:
```
iPhone poor iPhoneX

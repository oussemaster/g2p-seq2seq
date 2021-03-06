[![Build Status](https://travis-ci.org/cmusphinx/g2p-seq2seq.svg?branch=master)](https://travis-ci.org/cmusphinx/g2p-seq2seq)

# Sequence-to-Sequence G2P toolkit

The tool does Grapheme-to-Phoneme (G2P) conversion using transformer model
from tensor2tensor toolkit [1]. A lot of approaches in sequence modeling and
transduction problems use recurrent neural networks. But, transformer model
architecture eschews recurrence and instead relies entirely on an attention
mechanism to draw global dependencies between input and output [2].

This implementation is based on python
[TensorFlow](https://www.tensorflow.org/tutorials/seq2seq/),
which allows an efficient training on both CPU and GPU.

## Installation

The tool requires TensorFlow at least version 1.5.0 and Tensor2Tensor with version 1.5.0 or higher. Please see the installation
[guide](https://www.tensorflow.org/install/)
for TensorFlow installation details, and details about the Tensor2Tensor installation see in [guide](https://github.com/tensorflow/tensor2tensor)


The g2p_seq2seq package itself uses setuptools, so you can follow standard installation process:

```
sudo python setup.py install
```

You can also run the tests

```
python setup.py test
```

The runnable script `g2p-seq2seq` is installed in  `/usr/local/bin` folder by default (you can adjust it with `setup.py` options if needed) . You need to make sure you have this folder included in your `PATH` so you can run this script from command line.

## Running G2P

A pretrained 3-layer transformer model with 256 hidden units is [available for download on cmusphinx website](https://sourceforge.net/projects/cmusphinx/files/G2P%20Models/g2p-seq2seq-cmudict.tar.gz/download).
Unpack the model after download. The model is trained on [CMU English dictionary](http://github.com/cmusphinx/cmudict)

```
wget -O g2p-seq2seq-cmudict.tar.gz https://sourceforge.net/projects/cmusphinx/files/G2P%20Models/g2p-seq2seq-cmudict.tar.gz/download 
tar xf g2p-seq2seq-cmudict.tar.gz
```

The easiest way to check how the tool works is to run it the interactive mode and type the words

```
$ g2p-seq2seq --interactive --model_dir model_folder_path
...
> HELLO
...
INFO:tensorflow:HH EH L OW
...
>
```

To generate pronunciations for an English word list with a trained model, run

```
  g2p-seq2seq --decode your_wordlist --model_dir model_folder_path [--output decode_output_file_path]
```

The wordlist is a text file with one word per line

If you wish to list top N variants of decoding, set return_beams flag and specify beam_size:

```
  g2p-seq2seq --decode your_wordlist --model_dir model_folder_path --return_beams --beam_size number_returned_beams [--output decode_output_file_path]
```

To evaluate Word Error Rate of the trained model, run

```
  g2p-seq2seq --evaluate your_test_dictionary --model_dir model_folder_path
```

The test dictionary should be a dictionary in standard format:
```
HELLO HH EH L OW
BYE B AY
```

You may also calculate Word Error Rate considering all top N best decoded results. In this case we consider word decoding as error only if none of the decoded pronunciations will match with the ground true pronunciation of the word.

## Training G2P system

To train G2P you need a dictionary (word and phone sequence per line).
See an [example dictionary](http://github.com/cmusphinx/cmudict)

```
  g2p-seq2seq --train train_dictionary.dic --model_dir model_folder_path
```

You can set up maximum training steps:
```
  "--max_epochs" - Maximum number of training epochs (Default: 0).
     If 0 train until no improvement is observed
```

It is a good idea to play with the following parameters:
```
  "--size" - Size of each model layer (Default: 256).

  "--num_layers" - Number of layers in the model (Default: 3).

  "--filter_size" - The size of the filter layer in a convolutional layer (Default: 512)

  "--num_heads" - Number of applied heads in Multi-attention mechanism (Default: 4)
```

You can manually point out Development and Test datasets:
```
  "--valid" - Development dictionary (Default: created from train_dictionary.dic)
  "--test" - Test dictionary (Default: created from train_dictionary.dic)
```

If you need to continue train a saved model just point out the directory with the existing model:
```
  g2p-seq2seq --train train_dictionary.dic --model_dir model_folder_path
```

And, if you want to start training from scratch:
```
  "--reinit" - Rewrite model in model_folder_path
```

The differences in pronunciations between short and long words can be significant. So, seq2seq models applies bucketing technique to take account of such problems. On the other hand, splitting initial data into too many buckets can worse the final results. Because in this case there will be not enough amount of examples in each particular bucket. To get a better results, you may tune following three parameters that change number and size of the buckets:
```
  "--min_length_bucket" - the size of the minimal bucket (Default: 6)
  "--max_length" - maximal possible length of words or maximal number of phonemes in pronunciations (Default: 30)
  "--length_bucket_step" - multiplier that controls the number of length buckets in the data. The buckets have maximum lengths from min_bucket_length to max_length, increasing by factors of length_bucket_step (Default: 1.5)
```

After training the model, you may freeze it:
```
  g2p_seq2seq --model_dir model_folder_path --freeze
```

File "frozen_model.pb" will appeared in "model_folder_path" directory after launching previous command. And now, if you run one of the decoding modes, program will load and use this frozen graph.


#### Word error rate on CMU dictionary data sets

System | WER ([CMUdict PRONALSYL 2007](https://sourceforge.net/projects/cmusphinx/files/G2P%20Models/phonetisaurus-cmudict-split.tar.gz)), % | WER ([CMUdict latest\*](https://github.com/cmusphinx/cmudict)), %
--- | --- | ---
Baseline WFST (Phonetisaurus) | 24.4 | 33.89
Transformer num_layers=3, size=256   | 20.9 | 29.8
\* These results pointed out for dictionary without stress.

## References
---------------------------------------

[1] Lukasz Kaiser. "Accelerating Deep Learning Research with the Tensor2Tensor Library." In Google Research Blog, 2017.

[2] Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N. Gomez, Lucasz Kaiser, and Illia Polosukhin. "Attention Is All You Need."
arXiv preprint
arXiv:1706.03762, 2017.

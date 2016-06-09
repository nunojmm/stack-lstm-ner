## Transition-based NER system.

This system is part of a paper accepted at NAACL-HLT 2016 Conference.
See the paper here: http://arxiv.org/pdf/1603.01360v1.pdf

#### Desired labeling

    John Smith went to Pittsburgh .
     PER-----   O    O  LOC       O

Corresponding sequence of operations (generated by `convert-conll2trans.pl`)

    SHIFT
    SHIFT
    REDUCE(PER)
    OUT
    OUT
    SHIFT
    REDUCE(LOC)
    OUT

#### Data structures

 * **buffer** - sequence of tokens, read from left to right
 * **stack** - working memory
 * **output buffer** - sequence of labeled segments constructed from left to right

#### Operations

 * `SHIFT` - move word from **buffer** to top of **stack**
 * `REDUCE(X)` - all words on **stack** are popped, combined to form a segment and labeled with `X` and copied to **output buffer**
 * `OUT` - move one token from **buffer** to **output buffer**

#### Dataset & Preprocessing

We use the datasets from conll2002 and conll2003

Convert conll format to ner action (convert-conll2trans.pl) and convert it to parser friendly format (conll2parser.py).  

```bash
   perl convert-conll2trans.pl conll2003/train > conll2003/train.trans
   python conll2parser.py -f conll2003/train.trans > conll2003/train.parser 
```

 Link to the word vectors that we used in the NAACL 2016 paper for English:  [sskip.100.vectors](https://drive.google.com/file/d/0B8nESzOdPhLsdWF2S1Ayb1RkTXc/view?usp=sharing).


#### Build the system

The first time you clone the repository, you need to sync the cnn/ submodule.
```
git submodule init
git submodule update

mkdir build
cd build
cmake .. -DEIGEN3_INCLUDE_DIR=/path/to/eigen
make -j2
```

#### Training

    ./lstm-parse -T conll2003/train.parser -d conll2003/dev.parser --hidden_dim 100 --lstm_input_dim 100 -w sskip.100.vectors --pretrained_dim 100 --rel_dim 20 --action_dim 20 --input_dim 100 -t -S -D 0.3 > logNERYesCharNoPosYesEmbeddingsD0.3.txt &


### Decoding 


    ./lstm-parse -T conll2003/train.parser -d conll2003/test.parser --hidden_dim 100 --lstm_input_dim 100 -w sskip.100.vectors --pretrained_dim 100 --rel_dim 20 --action_dim 20 --input_dim 100 -m latest_model -S > output.txt
    python attach_prediction.py -p output.txt -t conll2003/test -o evaloutput.txt


#### Evaluation

Attach your prediction to test file

```bash
  python attach_prediction.py -p (prediction) -t /path/to/conll2003/test -o (output file)
  ./conlleval < (output file)
```

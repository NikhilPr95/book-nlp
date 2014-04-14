BookNLP
=======

Pipeline for processing books and other long documents, including:

* POS tagging (Stanford)
* Dependency Parsing (MaltParser)
* Character name clustering (e.g., "Tom", "Tom Sawyer", "Mr. Sawyer", "Thomas Sawyer" -> TOM_SAWYER)
* Quotation speaker identification
* Coreference (anaphora) resolution

How To Run
=======

The main executable class is novels/BookNLP.  `runjava` is a convenience wrapper for executing java (taking care of the classpath, memory, etc.).  To run, execute `./runjava novel/BooksNLP` with the relevant flags below. 


####Preliminaries

Download external jars (which are sadly too big for GitHub's 100MB file size limit)

* Download and unzip http://nlp.stanford.edu/downloads/stanford-corenlp-full-2013-11-12.zip
* copy stanford-corenlp-3.3.0-models.jar in that folder to the lib/ directory here


####Flags

-doc <text> : original text to process

-tok <file> : file path to save processed tokens to (or read them from, if it already exists)

-p : the path to write all diagnostic files to

-id : a unique book ID for this book

-printHTML	: print the text as an HTML file with character aliases, coref and speaker ID annotated

-f : force the processing of the original text file, even if the <file> in the -tok flag exists (if the -tok <file> exists, the process that would generate it is skipped)


####Example

    ./runjava novels/BookNLP -doc originalTexts/dickens.oliver.pg730.txt -printHTML -p output/dickens -id dickens.oliver.twist -tok tokens/dickens.oliver.tokens -f

(On a 2.6 GHz MBP, this takes about 3.5 minutes)

tokens/dickens.oliver.tokens contains the original book, one token per line, with part of speech, syntax and other annotations.  The (tab-separated) format is:

1. Paragraph id
2. Sentence id
3. Token id
4. Byte start
5. Byte end
6. Whitespace following the token (useful for pretty-printing the original text)
7. Syntax head id (-1 for the sentence root)
8. Original token
9. Normalized token (for quotes etc.)
10. Lemma
11. Penn Treebank POS tag
12. NER tag (PERSON, NUMBER, DATE, DURATION, MISC, TIME, LOCATION, ORDINAL, MONEY, ORGANIZATION, SET, O)
13. Stanford dependency label
14. Within-quotation flag

The output/dickens folder will now contain:

* dickens.oliver.twist.html (described above)
* dickens.oliver.twist.book (a representation of all of the characters' features, in JSON; this will be the input to our character model)

Modifying the code
================

With apache ant installed, running `ant` compiles everything.


Training coreference
====================

Coreference only needs to be trained when there's new training data (or new feature ideas: current features are based on syntactic tree distance, linear distance, POS identity, gender matching, quotation scope and salience).

####Data

Coreference annotated data is located in the coref/ directory. 

annotatedData.txt contains coreference annotations, in the (tab-separated) format:

1. book ID
2. anaphor token ID
3. antecendent token ID

bookIDs are mapped to their respective token files in docPaths.txt.  All of these token files are located in finalTokenData/.  These tokens files are all read-only -- since the annotations are keyed to specific token IDs in those files, we want to make sure they stay permanent.

####Training a model

Given the coref/ folder above, train new coreference weights with:

    ./runjava novels.training/TrainCoref -training coref/annotatedData.txt -o coref/weights.txt

-training specifies the input training file

-o specifies the output file to write the trained weights to

Two parameters control the amount of regularization in the model (higher regularization dampens the impact of any single feature, and L1 regularization removes features from the model; both help prevent overfitting to training data.)

-l1 specifies the L1 regularization parameter (higher = more weights end up driven to 0). Default = 2

-l2 specifies the L2 regularization parameter (higher = weights shrink faster). Default = .1

To use the newly trained weights in the pipeline above, copy them to files/coref.weights or specify them on the novels.BookNLP command line with the -w flag.

Quotation Id
============

Quotation/Speaker Id is currently deterministic (matching nearby character mentions); but with more annotated training data, this can become a log-linear model as well.

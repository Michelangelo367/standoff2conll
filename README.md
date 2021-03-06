# BRAT STANDOFF2CONLL 2004

An extended version of the original Sampo Pyysalo's conversion script that also supports conversion from brat-flavored standoff to the D. Roth and W. Yih CoNLL 2004 Relation Extractor training format.

This custom CONLL 2004 format is used to train the Stanford Relation Extractor model. An usage example can be seen in the https://github.com/aoldoni/tetre repository.


### USAGE

1. Annotate your text in [brat](http://brat.nlplab.org/index.html).

2. Generate Stanford NER training input - run the command targetting the folder with the annotated text (with the *.txt and *.ann files):  
  `./standoff2conll.py data/input/training/annotated/ > data/input/training/transformed/documents.tsv`

3. Prepare to generated the RE trianing input - prepare a columnar file with the POS tags of the raw annoated text. E.g.:
```
1	7	_	NUM	CD	_	0	ROOT	_	_
1	.	_	.	.	_	0	ROOT	_	_
1	RELATED	_	VERB	VBN	_	0	ROOT	_	_
1	WORK	_	VERB	VB	_	0	ROOT	_	_
1	LSH	_	NOUN	NNP	_	0	ROOT	_	_
1	functions	_	NOUN	NNS	_	0	ROOT	_	_
1	are	_	VERB	VBP	_	0	ROOT	_	_
1	ﬁ	_	.	NFP	_	0	ROOT	_	_
1	rst	_	ADV	RB	_	0	ROOT	_	_
1	introduced	_	VERB	VBN	_	0	ROOT	_	_
1	for	_	ADP	IN	_	0	ROOT	_	_
```
Starting from column 0, note that column 1 contains the token text and column 4 contains its POS tag. The file above can be generated as an output from e.g.: the Google's Syntaxnet Tensorflow model. For an example, see how [TETRE prepares such file](https://github.com/aoldoni/tetre/blob/develop/lib/brat_to_stanford/train.py).

4. Using the above file, generate Stanford RE training input - run the command targetting the folder with the annotated text (with the *.txt and *.ann files):  
  `./standoff2conll.py data/input/training/annotated/ --process ROTHANDYIH --process_pos_tag_input path/to/postagfile.tsv > data/input/training/transformed/documents.corp`

5. Text in the conll2004 format is now in the tsv file. It can now be used to train the Stanford Relation Extractor. NOTE: if you use custom entities/relations, please see the [section regarding custom entities](#custom-entities-in-stanford-re-training) below.


### THE CONLL 2004 FORMAT

The format is described [here](http://cogcomp.cs.illinois.edu/page/resource_view/43) with an example [here](http://cogcomp.cs.illinois.edu/Data/ER/conll04.corp). See an example below where the relation triplet Work_For(Bruno/Pusterla, Italian/Agricultural/Confederation) is denoted.

```
95	O	0	O	``	`	O	O	O
95	O	1	O	``	`	O	O	O
95	O	2	O	IN	If	O	O	O
95	O	3	O	PRP	it	O	O	O
95	O	4	O	VBZ	does	O	O	O
95	O	5	O	RB	not	O	O	O
95	O	6	O	NN	snow	O	O	O
95	O	7	O	,	COMMA	O	O	O
95	O	8	O	CC	and	O	O	O
95	O	9	O	DT	a	O	O	O
95	O	10	O	NN	lot	O	O	O
95	O	11	O	,	COMMA	O	O	O
95	O	12	O	IN	within	O	O	O
95	O	13	O	DT	this	O	O	O
95	O	14	O	NN	month	O	O	O
95	O	15	O	PRP	we	O	O	O
95	O	16	O	MD	will	O	O	O
95	O	17	O	VB	have	O	O	O
95	O	18	O	DT	no	O	O	O
95	O	19	O	NN	water	O	O	O
95	O	20	O	TO	to	O	O	O
95	O	21	O	VB	submerge	O	O	O
95	Other	22	O	CD/,/CD/NNS	150/COMMA/000/hectares	O	O	O
95	O	23	O	-LRB-	-LRB-	O	O	O
95	Other	24	O	CD/,/CD/NNS	370/COMMA/500/acres	O	O	O
95	O	25	O	-RRB-	-RRB-	O	O	O
95	O	26	O	IN	of	O	O	O
95	Other	27	O	NN	rice	O	O	O
95	O	28	O	,	COMMA	O	O	O
95	O	29	O	''	'	O	O	O
95	O	30	O	''	'	O	O	O
95	O	31	O	VBD	said	O	O	O
95	Peop	32	O	NNP/NNP	Bruno/Pusterla	O	O	O
95	O	33	O	,	COMMA	O	O	O
95	O	34	O	DT	a	O	O	O
95	O	35	O	JJ	top	O	O	O
95	O	36	O	NN	official	O	O	O
95	O	37	O	IN	of	O	O	O
95	O	38	O	DT	the	O	O	O
95	Org	39	O	JJ/NNP/NNP	Italian/Agricultural/Confederation	O	O	O
95	O	40	O	.	.	O	O	O

32	39	Work_For
```


### CUSTOM ENTITIES IN STANFORD RE TRAINING

By default Relation Extractor only accepts a few core entities, such as PEOPLE, ORGANISATION and LOCATION. If you attempt different entities you will see the following error:
```
Caused by: java.lang.RuntimeException: Cannot normalize ner tag Concept
	at edu.stanford.nlp.ie.machinereading.domains.roth.RothCONLL04Reader.getNormalizedNERTag(RothCONLL04Reader.java:85)
	at edu.stanford.nlp.ie.machinereading.domains.roth.RothCONLL04Reader.readSentence(RothCONLL04Reader.java:148)
	at edu.stanford.nlp.ie.machinereading.domains.roth.RothCONLL04Reader.read(RothCONLL04Reader.java:56)
	at edu.stanford.nlp.ie.machinereading.GenericDataSetReader.parse(GenericDataSetReader.java:132)
	... 3 more
```

To use your custom entities you have a few options:

1) Hard-code the new list of entities in the already hard-coded list inside the machine reading code, as per [this commit](https://github.com/aoldoni/stanford-corenlp/commit/ae6782266cb40629aefcecbab397da47f808abc3).

2) Use [this](https://github.com/aoldoni/stanford-corenlp/) fork I prepared of the Stanford CoreNLP. In this fork a new `possibleEntities` paremeter acceptes a comma separated list of entities for normalization.

3) Or wait until [support is added](https://github.com/stanfordnlp/CoreNLP/issues/359).

# LICENSE

Copyright (c) 2016, 2017 Alisson Oldoni, Sampo Pyysalo [MIT License](LICENSE)
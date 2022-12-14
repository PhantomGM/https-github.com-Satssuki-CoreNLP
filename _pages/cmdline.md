---
title: Using Stanford CoreNLP from the command line
keywords: command-line
permalink: '/cmdline.html'
---

## Quick start

The minimal command to run Stanford CoreNLP from the command line is:

```sh
java -cp "*" -Xmx2g edu.stanford.nlp.pipeline.StanfordCoreNLP -annotators tokenize,ssplit,pos,lemma,ner,parse,dcoref -file input.txt
```

If this command is run from the distribution directory, it processes the included [sample file](files/input.txt) `input.txt`. We use a wildcard `"*"` after `-cp` to load all jar files in the current directory &ndash; it needs to be in quotes. This command writes the output to an XML [file](files/input.txt.xml.txt) named `input.txt.out` in the same directory.

###  Notes

* Processing a short text like this is very inefficient. It takes a minute to load everything before processing begins. You should batch your processing.
* Stanford CoreNLP requires Java version 1.8 or higher.
* Specifying memory: `-Xmx2g` specifies the amount of RAM that Java will make available for CoreNLP. On a 64-bit machine, Stanford CoreNLP typically requires 2GB to run (and it may need even more, depending on the size of the document to parse). On a 32 bit machine (in 2016, this is most commonly a 32-bit Windows machine), you cannot allocate 2GB of RAM; probably you should try with `-Xmx1800m`. You are probably okay with a minimum of `-Xmx1500m`, but this amount of memory is a bit marginal. You probably can't run some annotators, such as the statistical `coref`.
* Stanford CoreNLP includes an interactive shell for analyzing sentences. If you do not specify any properties that load input files, you will be placed in the interactive shell. Type `q` to exit.

## Classpath

Your command line has to load the code, libraries, and model jars that CoreNLP uses. These are all contained in JAR files (compressed archives with extension ".jar") which come in the CoreNLP download or which can be downloaded on demand from Maven Central. The easiest way to do that is with a command line this, where `/Users/me/corenlp/` should be changed to the path where you put CoreNLP:

```sh
java -cp "/Users/me/corenlp/*" -Xmx2g edu.stanford.nlp.pipeline.StanfordCoreNLP -file inputFile
```

Alternatively, you can add this path to your CLASSPATH environment variable, so these libraries are always available.  

The "*" (which must be enclosed in quotes) says to add all JAR files in the given directory to the classpath. 
You can also individually specify the needed jar files. Use the following sort of command line, adjusting the JAR file date extensions `VV` to your downloaded release.

```sh
java -cp stanford-corenlp-VV.jar:stanford-corenlp-VV-models.jar:xom.jar:joda-time.jar:jollyday.jar:ejml-VV.jar -Xmx2g edu.stanford.nlp.pipeline.StanfordCoreNLP -file inputFile
```

The command above works for Mac OS X or Linux. For Windows, the colons (:) separating the jar files need to be semi-colons (;). If you are not sitting in the distribution directory, you'll also need to include a path to the files before each.

## Configuration

Before using Stanford CoreNLP, it is usual to create a configuration file (a Java Properties file). Minimally, this file should contain the "annotators" property, which contains a comma-separated list of Annotators to use. For example, the setting below enables: tokenization, sentence splitting (required by most Annotators), POS tagging, lemmatization, NER, syntactic parsing, and coreference resolution.

> annotators = tokenize, ssplit, pos, lemma, ner, parse, dcoref

For example, using the properties file [sampleProps.properties](files/sampleProps.properties), you can run using these properties as follows:

```sh
java -cp "*" -Xmx2g edu.stanford.nlp.pipeline.StanfordCoreNLP -props sampleProps.properties
```

This results in the output file [input.txt.output](files/input.txt.output) given the same input file `input.txt`.

However, if you just want to specify one or two properties, you can instead place them on the command line, as in the first example, where annotators were specified in the command.

```sh
java -cp "*" -Xmx2g edu.stanford.nlp.pipeline.StanfordCoreNLP -annotators tokenize,ssplit,pos,lemma,ner -file input.txt
```

The `-props` parameter is optional. By default, Stanford CoreNLP will search for StanfordCoreNLP.properties in your classpath and use the defaults included in the distribution.

The `-annotators` argument is actually optional. If you leave it out, the code uses a built in properties file, which enables the following annotators: tokenization and sentence splitting, POS tagging, lemmatization, NER, parsing, and coreference resolution (that is, what we used in these examples).

If you have a lot of text but all you want to do is to, say, get part-of-speech (POS) tags, then you should definitely specify an annotators list, since you can then omit later annotators which invoke much more expensive processing that you don't need. For example, you might give the command:

```sh
java -cp "*" -Xmx500m edu.stanford.nlp.pipeline.StanfordCoreNLP -annotators tokenize,ssplit,pos -file wikipedia.txt -outputFormat conll
```

We provide a small shell script `corenlp.sh`. On Linux or OS X, this may be useful in allowing you to type shorter command lines to invoke CoreNLP. For example, you can instead say:

```sh
./corenlp.sh -annotators tokenize,ssplit,pos -file wikipedia.txt -outputFormat conll
```


## Languages other than English

You first have to have available a models jar file for the language you wish to use. You can download it from this site or you can use
the models file on [Maven Central](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22stanford-corenlp%22). You add it to your pom file like this:

```xml
<dependency>
    <groupId>edu.stanford.nlp</groupId>
    <artifactId>stanford-corenlp</artifactId>
    <version>3.6.0</version>
    <classifier>models-chinese</classifier>
</dependency>
```

Our examples assume that you are in the root directory of CoreNLP and that these extra jar files are also available there. Each language jar contains a default properties file for the appropriate language. Working with text in another language is then as easy as specifying this properties file. For example, for Chinese:

```sh
java -mx3g -cp "*" edu.stanford.nlp.pipeline.StanfordCoreNLP -props StanfordCoreNLP-chinese.properties -file chinese.txt -outputFormat text
```

You can as usual specify details on the annotators and properties:

```sh
java -mx1g -cp "*" edu.stanford.nlp.pipeline.StanfordCoreNLP -props StanfordCoreNLP-french.properties -annotators tokenize,ssplit,pos,depparse -file french.txt -outputFormat conllu
```

## Input

To process one file, use the `-file` option.  To process a list of files, use the `-filelist` parameter:

```sh
java -cp "*" -Xmx2g edu.stanford.nlp.pipeline.StanfordCoreNLP [ -props myprops.props ] -filelist filelist.txt
```

where the `-filelist` parameter points to a file whose content lists all files to be processed (one per line).

If you do not specify any properties that load input files, you will be placed in the [interactive shell](repl.html). Type `q` to exit.

If your input files have XML tags in them, you may wish to use the `cleanxml` annotator to preprocess it.

### Inputting serialized files

If (and only if) the input filename ends with ".ser.gz" then CoreNLP will interpret the file as the output of a previous annotation run, to which you presumably want to add on further annotations. CoreNLP will read these Annotations using the class specified in the `inputSerializer` property. The options for this are the same as for `outputSerializer` [below](cmdline.html#output-serializer). Note: To successfully load a pipeline for layering on additional annotations, you need to include the property `enforceRequirements = false` to avoid complaints about required earlier annotators not being present in the pipeline.


## Output

For each input file, Stanford CoreNLP generates one output file, with a name that adds an extra extension to the input filename. This output may contain the output of all annotations that were done, or just a subset of them. For example, for the above configuration and a file containing the text below:

> Stanford University is located in California. It is a great university.

Stanford CoreNLP generates [this output](files/input.txt.output).

Note that this XML output can use the `CoreNLP-to-HTML.xsl` stylesheet file, which comes with the CoreNLP download or can be downloaded from [here](files/CoreNLP-to-HTML.xsl). This stylesheet enables human-readable display of the above XML content. For example, the previous example should be displayed like [this](files/input.txt.xml).

### Output options

The following properties are associated with output :

* `-outputDirectory` : By default, output files are written to the current directory. You may specify an alternate output directory with the flag `-outputDirectory`. 
* `-outputExtension` : Output filenames are the same as input filenames but with `-outputExtension` added to them (`.xml` by default). 
* `-noClobber` : By default, files are overwritten (clobbered). Pass `-noClobber` to avoid this behavior. 
* `-replaceExtension` : If you'd rather replace the extension with the `-outputExtension`, pass the `-replaceExtension` flag. This will result in filenames like `input.xml` instead of `input.txt.xml` (when given `input.txt` as an input file).
* `-outputFormat` : Different methods for outputting results.  Can be:
  * "text": An ad hoc human-readable text format. Tokens, s-expression parse trees, relation(head, dep) dependencies. Output file extension is `.out`. This is the default output format only if the XMLOutputter is unavailable.
  * "xml": An XML format with accompanying XSLT stylesheet, which allows web browser rendering. Output file extension is `.xml`.  This is the default output format, unless the XMLOutputter is unavailable.
  * "json": JSON. Output file extension is `.json`. 'Nuf said.
  * "conll": A tab-separated values (TSV) format. Output extension is `.conll`. This output format usually only gives a partial view of an `Annotation` and doesn't correspond to any particular CoNLL format. By default, the columns written are: idx, word, lemma, pos, ner, headidx, deprel. You can customize which fields are written with the `outputFormatOptions` property. Its value is a comma separated list of output key names, where the names are ones understood by `AnnotationLookup.KeyLookup`. Available names include the seven used in the default output and others such as shape, speaker. You can for instance write out just tokenized text, with one token per line and a blank line between sentences by using `-outputFormatOptions word`.
  * "conllu": [CoNLL-U](https://universaldependencies.github.io/docs/format.html) output format, another tab-separated values (TSV) format.  Output extension is `.conllu`. This representation may give only a partial view of an `Annotation`.
  * "serialized": Produces some serialized version of each `Annotation`. May or may not be lossy. What you actually get depends on the `outputSerializer` property, which you should also set. The default is the `GenericAnnotationSerializer`, which uses the built-in Java object serialization and writes a file with extension `.ser.gz`.

### Output serializer

The value of the `outputSerializer` property is the name of a class which extends `edu.stanford.nlp.pipeline.AnnotationSerializer`. Valid choices include:
`edu.stanford.nlp.pipeline.GenericAnnotationSerializer`,
`edu.stanford.nlp.pipeline.CustomAnnotationSerializer`,
`edu.stanford.nlp.pipeline.ProtobufAnnotationSerializer`;
`edu.stanford.nlp.kbp.common.KBPProtobufAnnotationSerializer`,
`edu.stanford.nlp.kbp.slotfilling.ir.index.KryoAnnotationSerializer`. If unspecified the value of the `serializer` property will be tried instead. If it is also not defined, the default is to use `edu.stanford.nlp.pipeline.GenericAnnotationSerializer`.

The `ProtobufAnnotationSerializer` uses the Java methods writeDelimitedTo() and parseDelimitedFrom(), which allow sending several length-prefixed messages in one stream. Unfortunately, Google has declined to implement these methods for Python or C++. You can get information from Stack Overflow and other places on how to roll your own version for C++ or Python. Probably the best place is [here](http://stackoverflow.com/questions/2340730/are-there-c-equivalents-for-the-protocol-buffers-delimited-i-o-functions-in-ja/) but there are many other sources of information including: [here](http://stackoverflow.com/questions/8269452/google-protocol-buffers-parsedelimitedfrom-and-writedelimitedto-for-c), [here](https://github.com/google/protobuf/pull/710),  [here](http://stackoverflow.com/questions/11484700/python-example-for-reading-multiple-protobuf-messages-from-a-stream), and [here](http://eli.thegreenplace.net/2011/08/02/length-prefix-framing-for-protocol-buffers).
[This Stack Overflow question](http://stackoverflow.com/questions/39433279/read-protobuf-serialization-of-stanfordnlp-output-in-python/40964310) explicitly addresses the issue for CoreNLP.

### A note on numbering

In all output formats (and in our code), we number sentences and character offsets from 0 and we number tokens from 1. We realize that this is inconsistent! However, it seemed to be the best thing to do. Numbering character offsets from 0 is the only good choice, given how the Java String class and most modern programming languages, which followed Dijkstra's arguments for indexing from 0, work. Numbering tokens from 1 not only corresponds to the human-natural convention (???the first word of the sentence???) but most importantly is consistent with common NLP standards, such as the CoNLL and [CoNLL-U](http://universaldependencies.org/format.html) formats, which number tokens starting from 1.


## Character encoding

CoreNLP???s default character encoding is Unicode's UTF-8. You can change the encoding used by supplying the program with the command line flag `-encoding FOO` (or including the corresponding property in a properties file that you are using). We???ve done a lot of careful work to make sure CoreNLP works with any character encoding supported by Java. Want to use ISO-8859-15 or GB18030? Be our guest!

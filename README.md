# Lucene Analyzers for Buddhist Chinese

This repository contains bricks to process Chinese in Lucene, mainly:
- stopwords
- SC/TC conversion
- Hanzi variants
- Hanzi to Pinyin conversion
- Pinyin syllable tokenizer

## Installation

You can install this analyzer from Maven:

```xml
    <dependency>
      <groupId>io.bdrc.lucene</groupId>
      <artifactId>lucene-zh</artifactId>
      <version>0.1.0</version>
    </dependency>
```

## Building from source

A compiled Trie is needed in order to build a complete jar, the base command line to build a jar is thus:

```
mvn clean compile exec:java package
```

The following options alter the packaging:

- `-DincludeDeps=true` includes `io.bdrc.lucene:stemmer` in the produced jar file
- `-DperformRelease=true` signs the jar file with gpg

## Indexing Pipeline

```
TC ⟾ normalized TC ⟾ SC ⟾ normalized SC ⟾ normalized PY_strict ⟾ normalized PY_lazy
```

`normalized TC/SC`: any combination of the following treatments: synonyms, alternatives and stopwords.

`normalized PY`: PY is lower-cased and split in syllables (to match the general policy of indexing individual ideograms). We call `PY_strict` Pinyin with tone indication (diacritics or numbers) and `PY_lazy` Pinyin with no tone indication.

## Constructors

```
ChineseAnalyzer(String indexEncoding,String inputEncoding,boolean stopwords, int variants) 
    indexEncoding- "TC", "SC", "PY_strict" or "PY_lazy"
    inputEncoding- "TC", "SC", "PY_strict" or "PY_lazy"
    stopWords    - true to filter stopwords, false otherwise
    variants     - 0: no variant; 1: synonyms; 2: alternatives; 3: both
```

```
ChineseAnalyzer(String profile)
```

| Profiles            | inputEncoding | indexEncoding | stopWords | variants |
| :------------------ | :------------ | :------------ | :-------- | :------: |
| `exactTC`          | TC            | TC            | false     | 0        |
| `TC`               | TC            | TC            | true      | 3        |
| `TC2SC`            | TC            | SC            | true      | 3        |
| `TC2PYstrict`     | TC            | PYstrict      | true      | 3        |
| `TC2PYlazy`       | TC            | PYlazy        | true      | 3        |
| `SC`               | SC            | SC            | true      | 3        |
| `SC2PYstrict`     | SC            | PYstrict      | true      | 3        |
| `SC2PYlazy`       | SC            | PYlazy        | true      | 3        |
| `PYstrict`        | PYstrict      | PYstrict      | false     | 0        |
| `PYstrict2PYlazy`| PYstrict      | PYlazy        | false     | 0        |
| `PYlazy`          | PYlazy        | PYlazy        | false     | 0        |


## Components

### Tokenizers

#### StandardTokenizer

Produces ideogram-based tokens(it incorporated the historical Chinese Tokenizer). 

#### WhitespaceTokenizer

Used together with `PinyinSyllabifyingFilter` in order to avoid giving it big strings.

### Filters

#### PinyinNormalizingFilter (MappingCharFilter)

TODO: when we have more pinyin data or when we know how users type their queries, assess if the normalization is sufficient or not.

#### TC2SCFilter (TokenFilter)

Leverages Unihan data to replace token content with the SC equivalent.

#### ZhToPinyinFilter (TokenFilter)

Replaces the token content(TC and SC) with the pinyin transcription. 

#### LazyPinyinFilter (TokenFilter)

Removes tone marks in Pinyin.

#### LowerCaseFilter (MappingCharFilter)

Used as a pre-processing step for PY indexing

#### PinyinSyllabifyingFilter (TokenFilter)

Produces syllable-based tokens using `PinyinAlphabetTokenizer`.
Supports both strict and lazy pinyin.

#### ZhSynonymsFilter (MappingCharFilter)

Leverages Unihan's kSemanticVariant field to index the same variant for all synonyms.

#### ZhAlternatesFilter (MappingCharFilter)

Leverages Unihan's kZVariant field to index the same variant for stylistic variants of the same ideogram.

## Sizes

In the Unihan database for Unicode 10, 88884 codepoints have an entry (adding full ideograms and parts of surrogate pairs).
82829 entries have no information about being TC nor SC, 3037 are specifically TC, 3007 are specifically SC and 11 have information about both TC and SC.

There are 1655 possible syllables in PY and 469 in PY with no diacritics.

## Resources

`src/main/resources` is the output of lucene-zh-data, generated by `make`, except for `pinyin-alphabet.dict`, coming from [here](https://github.com/medcl/elasticsearch-analysis-pinyin/tree/master/src/main/resources).

`src/main/resources/zh-stopwords.txt` is [this stop-list](https://github.com/stopwords-iso/stopwords-zh/blob/master/stopwords-zh.txt) 

`src/main/resources/zh-stopwords_analyzed.txt` is the same list as above with the corresponding SC, PYstrict and PYlazy corresponding strings. It was generated using `PrettyPrintResult.java`.

## Licence
The code is Copyright 2018 Buddhist Digital Resource Center, and is provided under [Apache License 2.0](LICENSE).

Files in `src/main/resources` remain under [Unicode License](http://unicode.org/copyright.html), except `zh-stopwords.txt`, under [MIT Licence](https://opensource.org/licenses/MIT) and `pinyin-alphabet.dict`, under [Apache 2 Licence](LICENCE).

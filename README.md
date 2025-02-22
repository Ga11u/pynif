# pynif [![Python tests](https://github.com/wetneb/pynif/actions/workflows/ci.yml/badge.svg)](https://github.com/wetneb/pynif/actions/workflows/ci.yml) [![Coverage Status](https://coveralls.io/repos/github/wetneb/pynif/badge.svg?branch=master)](https://coveralls.io/github/wetneb/pynif?branch=master) [![PyPI version](https://img.shields.io/pypi/v/pynif.svg)](https://pypi.org/project/pynif/)

The [NLP Interchange Format (NIF)](http://persistence.uni-leipzig.org/nlp2rdf/) is an RDF/OWL-based format that aims to achieve interoperability between Natural Language Processing (NLP) tools, language resources and annotations. It offers a standard representation of annotated texts for tasks such as [Named Entity Recognition](https://en.wikipedia.org/wiki/Named-entity_recognition) or [Entity Linking](https://en.wikipedia.org/wiki/Entity_linking). It is used by [GERBIL](https://github.com/dice-group/gerbil) to run reproducible evaluations of annotators.

This Python library can be used to serialize and deserialized annotated corpora in NIF.

## Documentation

[NIF Documentation](http://persistence.uni-leipzig.org/nlp2rdf/)

## Supported NIF versions

NIF 2.1, serialized in [any of the formats supported by rdflib](https://rdflib.readthedocs.io/en/stable/plugin_parsers.html)

## Overview

This library is revolves around three core classes:
 * a `NIFContext` is a document (a string);
 * a `NIFPhrase` is the annotation of a snippet of text (usually a phrase) in a document;
 * a `NIFCollection` is a set of documents, which constitutes a collection.
In NIF, each of these objects is identified by a URI, and their attributes and relations are encoded by RDF triples between these URIs.
This library abstracts away the encoding by letting you manipulate collections, contexts and phrases as plain Python objects.

## Quick start

Install pynif with `pip install pynif`.

0) Import and create a collection

```python
from pynif import NIFCollection

collection = NIFCollection(uri="http://freme-project.eu")
```

1) Create a context

```python
context = collection.add_context(
    uri="http://freme-project.eu/doc32",
    mention="Diego Maradona is from Argentina.")

```

2) Create entries for the entities

```python
context.add_phrase(
    beginIndex=0,
    endIndex=14,
    taClassRef=['http://dbpedia.org/ontology/SportsManager', 'http://dbpedia.org/ontology/Person', 'http://nerd.eurecom.fr/ontology#Person'],
    score=0.9869992701528016,
    annotator='http://freme-project.eu/tools/freme-ner',
    taIdentRef='http://dbpedia.org/resource/Diego_Maradona',
    taMsClassRef='http://dbpedia.org/ontology/SoccerManager')

context.add_phrase(
    beginIndex=23,
    endIndex=32,
    taClassRef=['http://dbpedia.org/ontology/PopulatedPlace', 'http://nerd.eurecom.fr/ontology#Location',
    'http://dbpedia.org/ontology/Place'],
    score=0.9804963628413852,
    annotator='http://freme-project.eu/tools/freme-ner',
    taMsClassRef='http://dbpedia.org/resource/Argentina')
```

3) Finally, get the output with the format that you need

```python
generated_nif = collection.dumps(format='turtle')
print(generated_nif)
```

You will obtain the NIF representation as a string:
```turtle
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix nif: <http://persistence.uni-leipzig.org/nlp2rdf/ontologies/nif-core#> .
@prefix itsrdf: <http://www.w3.org/2005/11/its/rdf#> .
@prefix dcterms: <http://purl.org/dc/terms/>

<http://freme-project.eu> a nif:ContextCollection ;
    nif:hasContext <http://freme-project.eu/doc32> ;
    dcterms:conformsTo <http://persistence.uni-leipzig.org/nlp2rdf/ontologies/nif-core/2.1> .

<http://freme-project.eu/doc32> a nif:Context,
        nif:OffsetBasedString ;
    nif:beginIndex "0"^^xsd:nonNegativeInteger ;
    nif:endIndex "33"^^xsd:nonNegativeInteger ;
    nif:isString "Diego Maradona is from Argentina." .

<http://freme-project.eu/doc32#offset_0_14> a nif:OffsetBasedString,
        nif:Phrase ;
    nif:anchorOf "Diego Maradona" ;
    nif:beginIndex "0"^^xsd:nonNegativeInteger ;
    nif:endIndex "14"^^xsd:nonNegativeInteger ;
    nif:referenceContext <http://freme-project.eu/doc32> ;
    nif:taMsClassRef <http://dbpedia.org/ontology/SoccerManager> ;
    itsrdf:taAnnotatorsRef <http://freme-project.eu/tools/freme-ner> ;
    itsrdf:taClassRef <http://dbpedia.org/ontology/Person>,
        <http://dbpedia.org/ontology/SportsManager>,
        <http://nerd.eurecom.fr/ontology#Person> ;
    itsrdf:taConfidence 9.869993e-01 ;
    itsrdf:taIdentRef <http://dbpedia.org/resource/Diego_Maradona> .

<http://freme-project.eu/doc32#offset_23_32> a nif:OffsetBasedString,
        nif:Phrase ;
    nif:anchorOf "Argentina" ;
    nif:beginIndex "23"^^xsd:nonNegativeInteger ;
    nif:endIndex "32"^^xsd:nonNegativeInteger ;
    nif:referenceContext <http://freme-project.eu/doc32> ;
    nif:taMsClassRef <http://dbpedia.org/resource/Argentina> ;
    itsrdf:taAnnotatorsRef <http://freme-project.eu/tools/freme-ner> ;
    itsrdf:taClassRef <http://dbpedia.org/ontology/Place>,
        <http://dbpedia.org/ontology/PopulatedPlace>,
        <http://nerd.eurecom.fr/ontology#Location> ;
    itsrdf:taConfidence 9.804964e-01 .
    
```

4) You can then parse it back:

```python
parsed_collection = NIFCollection.loads(generated_nif, format='turtle')

for context in parsed_collection.contexts:
   for phrase in context.phrases:
       print(phrase)
```

## Supported NIF OffsetBasedString

A context can be represented by an OffsetBasedString URI or a ContextHashBasedString URI. The ContextHashBasedString URI format is discussed in the paper Linked-Data Aware URI Schemes for Referencing Text Fragments (https://doi.org/10.1007/978-3-642-33876-2_17) page 4. 

To use ContextHashBasedString URIs, **you must provide them when creating Contexts and Phrases. The current pynif does not automaticaly create them for you** as it does with the OffsetBasedString.
To provide a ContextHashBasedString URI instead of a OffsetBasedString URI, you must set the ``:param: is_hash_based_uri`` to ``True`` (by default ``is_hash_based_uri`` is ``False`` and the pynif works with ``nif:OffsetBasedString``). See the following examples:

```py
context = NIFContext(
    uri='http://freme-project.eu#hash_0_33_cf35b7e267d05b7ca8aba0651641050b_Diego%20Maradona%20is%20fr',
    is_hash_based_uri = True,
    mention="Diego Maradona is from Argentina.")

context.add_phrase(
    uri='http://freme-project.eu#hash_10_14_d18575292bcf716916eb99eb8927377f_Diego%20Maradona',
    is_hash_based_uri = True,
    beginIndex=0,
    endIndex=14,
    score=0.9869992701528016,
    taClassRef=['http://dbpedia.org/ontology/SportsManager', 
        'http://dbpedia.org/ontology/Person', 
        'http://nerd.eurecom.fr/ontology#Person'],
    annotator='http://freme-project.eu/tools/freme-ner',
    taIdentRef='http://dbpedia.org/resource/Diego_Maradona',
    taMsClassRef='http://dbpedia.org/ontology/SoccerManager')
```

The output in TURTLE format:

```python
generated_nif = context.dumps(format='turtle')
print(generated_nif)
```
```TURTLE
@prefix xsd:   <http://www.w3.org/2001/XMLSchema#> .
@prefix itsrdf: <http://www.w3.org/2005/11/its/rdf#> .
@prefix nif:   <http://persistence.uni-leipzig.org/nlp2rdf/ontologies/nif-core#> .
                
<http://freme-project.eu#hash_0_33_cf35b7e267d05b7ca8aba0651641050b_Diego%20Maradona%20is%20fr>
    a nif:ContextHashBasedString , nif:Context ;
    nif:beginIndex  "0"^^xsd:nonNegativeInteger ;
    nif:endIndex    "33"^^xsd:nonNegativeInteger ;
    nif:isString    "Diego Maradona is from Argentina." .

<http://freme-project.eu#hash_10_14_d18575292bcf716916eb99eb8927377f_Diego%20Maradona>
    a nif:ContextHashBasedString, nif:Phrase ;
    nif:anchorOf "Diego Maradona" ;
    nif:beginIndex "0"^^xsd:nonNegativeInteger ;
    nif:endIndex "14"^^xsd:nonNegativeInteger ;
    nif:referenceContext <http://freme-project.eu#hash_0_33_cf35b7e267d05b7ca8aba0651641050b_Diego%20Maradona%20is%20fr> ;
    nif:taMsClassRef <http://dbpedia.org/ontology/SoccerManager> ;
    itsrdf:taAnnotatorsRef <http://freme-project.eu/tools/freme-ner> ;
    itsrdf:taClassRef <http://dbpedia.org/ontology/Person>, 
        <http://dbpedia.org/ontology/SportsManager>, 
        <http://nerd.eurecom.fr/ontology#Person> ;
    itsrdf:taConfidence 9.869993e-01 ;
    itsrdf:taIdentRef <http://dbpedia.org/resource/Diego_Maradona> .
```

**URIs formatted as ContextHashBasedString can be created using the companion algortihm from pynif.tools** 

```python 
from pynif.tools import context_hash_based_string

text = "Diego Maradona is from Argentina."
original_uri = "http://freme-project.eu#"

contextHashBasedString = context_hash_based_string(text, original_uri)
#Output = http://freme-project.eu#hash_0_33_cf35b7e267d05b7ca8aba0651641050b_Diego%20Maradona%20is%20fr

#To annotated an occurence with beginIndex and endIndex
contextHashBasedString = context_hash_based_string(text, original_uri,beginIndex=0, endIndex=14,context_length=10)
#Output = http://freme-project.eu#hash_10_14_d18575292bcf716916eb99eb8927377f_Diego%20Maradona

#To annotated an occurence with beginIndex and length
contextHashBasedString = context_hash_based_string(text, original_uri,beginIndex=0, length=14)
#Output = http://freme-project.eu#hash_10_14_d18575292bcf716916eb99eb8927377f_Diego%20Maradona

#An extension of the contextHashBasedString algorithm to ensure uniqueness
contextHashBasedString = context_hash_based_string(text, original_uri,beginIndex=0, endIndex=14,context_length=10, safe=True)
#Output = http://freme-project.eu#hash_10_14_d18575292bcf716916eb99eb8927377f_Diego%20Maradona_0_14'
```

The `text` and `orginal_uri` are required parameters, the rest are optional.

## Issues

If you have any problems with or questions about this library, please contact us through a [GitHub issue](https://github.com/wetneb/pynif/issues).

## Releasing a new version

Make sure the version in `setup.py` is up to date, create and upload a git tag, and then:

```
python -m build --sdist
python -m build --wheel
python -m twine upload dist/*
```

Increment the version in `setup.py`.

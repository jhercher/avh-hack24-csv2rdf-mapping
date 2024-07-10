# ADA Hackathon avh-hack24 csv2rdf mapping

This is a short howto to for mapping the [Viral Texts (VT)](https://viraltexts.org/) dataset to an RDF representation.
VT has a huge collection of poems re-printed in serveral newspapers in the past centuries. Reprints of a poem are called whitnesses and they are organized in clusters.
Here we use a CSV of one cluster, found on VT's [elastic search frontend](https://orca-app-ibxg3.ondigitalocean.app/).
The goal is to make the data more self-explanatory following the principles of Linked Data.
The complete dataset of CSVs is 1/2 TB here we use the CSV of one cluster as an example.

Main Question:
what makes the csv to work with efficiently (how do scholars want to use it?) 

## Overview

- [Data](#data)
- [Mapping](#mapping)
- [RDF](#rdf)
- 

## Data

The [CSV](cluster_8591153933_data-wild.csv) of a cluster has the following header: 

```
Title,Paragraphs,Place,Date,Open,URL,Coverage,Images,Witness_id,Cluster_id 
```

- **Title**: The title of the newspaper in wich the poem was published.
- **Paragraphs**: The text of the poem, as recognized by the OCR software.
- **Place**: The place where the newspaper was published.
- **Date**: The date of the publication of the newspaper.
- **Open**: Whether the link to the Annotation in the newspaper is open or closed.
- **URL**: Link to view the digitized pages of the newspaper.
- **Coverage**: The place where the poem was published.
- **Images**: ?.
- **Witness_id**: The unique id of one printed poem 
- **Cluster_id**: The unique id of the cluster.


## Mapping
[Tarql](https://tarql.github.io/) is a good choice to map CSV to RDF. It is a command line tool and simply requires a SPAQL construct query to do what we want:

```sparql
# Mapping Example for a Cluster of Witnesses for a Poem in viraltexts.org 
BASE  <https://w3id.org/viraltexts/resource/> 
PREFIX oa: <http://www.w3.org/ns/oa#> 
PREFIX schema: <https://schema.org/> 
PREFIX prov: <http://www.w3.org/ns/prov#> #seeAlso: http://events.linkeddata.org/ldow2009/papers/ldow2009_paper18.pdf
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

CONSTRUCT { 
   ?iri a oa:Annotation ;
   oa:hasTarget ?URL ;
   dc:title ?Title ;
   dc:partOf ?Cluster_uri ;
   prov:createdBy <https://viraltexts.org/> ;
   oa:bodyValue ?Paragraphs ;
   .

}
WHERE { 
 # Bind URI of the Wittness (AnnotationObject)
 BIND(iri(?Witness_id) as ?iri) 
 # Bind URI of the Cluster (Collection of Annotations)
 BIND (URI(CONCAT('https://w3id.org/viraltexts/resource/', ?Cluster_id)) AS ?Cluster_uri)  
}
```  

The CSV can then be mapped to RDF using the following command:

```BASH
tarql mapping.sparql nameOfcluster.csv
```
The Output is a Turtle file with , i.e.

```turtle
    …

<https://w3id.org/viraltexts/resource/pDegbJABXJXPPepFNbin>
        rdf:type        oa:Annotation ;
        oa:hasTarget    "http://trove.nla.gov.au/ndp/del/article/115007520" ;
        dc:title        "Riverine Herald (Echuca, Vic. : Moama, NSW : 1869 - 1954)" ;
        dc:partOf       <https://w3id.org/viraltexts/resource/8591153933> ;
        prov:createdBy  <https://viraltexts.org/> ;
        oa:bodyValue    "Close Seasons for Game, &c. The\nfollowing are tho close seasons\nthroughout Victoria for tho year THE\nWHOLE YE&It, All birds, oxcept sparrows\nand minas, not in* igonouR to Australia\nand their 
        
        …
```

Add `--ntriples` option to tarql to get a NTriples file :

`tarql --ntriples mapping.sparql cluster_8591153933_data-wild.csv ` 

```ntriples
…
<https://w3id.org/viraltexts/resource/WjegbJABXJXPPepFNbin> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/ns/oa#Annotation> .
<https://w3id.org/viraltexts/resource/WjegbJABXJXPPepFNbin> <http://www.w3.org/ns/oa#hasTarget> "http://trove.nla.gov.au/ndp/del/article/114965971" .
<https://w3id.org/viraltexts/resource/WjegbJABXJXPPepFNbin> <http://purl.org/dc/elements/1.1/title> "Riverine Herald (Echuca, Vic. : Moama, NSW : 1869 - 1954)" .
<https://w3id.org/viraltexts/resource/WjegbJABXJXPPepFNbin> <http://purl.org/dc/elements/1.1/partOf> <https://w3id.org/viraltexts/resource/8591153933> .
…
```


## Mapping Choices
These mapping choices are preliminary and only used as an example. They might be changed in the future.

Each row of the CSV file can be referred to with its id (`Whitness_id`), therefore this is the Object of interest we can model as oa:Annotation of the [openannotation ontology](https://www.w3.org/ns/oa#).
The `oa:hasTarget` property is used to link the Annotation for viewing in the newspaper. `oa:bodyValue` contains the OCR text of each withness.
Dublin core is used to describe the metadata of the publication that contains the poem and its annotation, hence we use `dc:title` for the title of the newspaper and `dc:partOf` to link the Annotation to the cluster of annotations.
The provernance ontology may be used in the future to model provernance of the OCR and annotation  creation process. As of now  `prov:createdBy` only contains the URI of the viraltexts.org website.


## Cool URIs don't change

To create stable URIs the [w3id](https://w3id.org/) service can be used. Just follow the instructions on the [w3id website](https://w3id.org/) and create a .htaccess file to redirect requests to the https://w3id.org/viraltexts/resource/<id>` to the actual server.

 ```bash
RewriteEngine On
RewriteRule (.*) https://viraltexts.org/$1 [R=303,L,QSA]
```  


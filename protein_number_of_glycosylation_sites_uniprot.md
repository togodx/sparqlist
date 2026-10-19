# UniProt glycosylation-site count distribution

## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot/)
- Query
  - Input
    - UniProt ID
  - Output
    - The number of glycosylation site
- Creators
   - 守屋

## Endpoint

https://rdfportal.org/sib/sparql

## `withAnnotation`
```sparql
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?leaf ?label (COUNT (DISTINCT ?annotation) AS ?value)
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
  ?leaf a core:Protein ;
        core:mnemonic ?label ;
        core:organism taxon:9606 ;
        core:proteome ?proteome ;
        core:annotation ?annotation .
  ?annotation a core:Glycosylation_Annotation .
}
```

## `withoutAnnotation`
```sparql
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?leaf ?label ?value
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
  ?leaf a core:Protein ;
        core:mnemonic ?label ;
        core:organism taxon:9606 ;
        core:proteome ?proteome .
  MINUS {
    ?leaf core:annotation ?annotation .
    ?annotation a core:Glycosylation_Annotation .
  }
  BIND("0" AS ?value)
}
```

## `results`
```javascript
({withAnnotation, withoutAnnotation})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  
  return withAnnotation.results.bindings.concat(withoutAnnotation.results.bindings).map(d => {
    return {
      id: d.leaf.value.replace(idPrefix, ""),
      label: d.label.value,
      value: Number(d.value.value),
      binId: Number(d.value.value) + 1,
      binLabel: d.value.value
    }
  });
}
```
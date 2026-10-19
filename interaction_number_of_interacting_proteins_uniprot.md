# UniProt interacting human proteins (IntAct) count distribution

- human protein - human protein のみ
  - endpoint に human しか入っていないため
  - 本家 UniProt には virus protein との相互作用もあるが、RDF が種に依存してるので TogoSite endpoint では相手が取れない

## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot)
  - The number of interacting proteins from UniProt
  - This item based on the data of March, 2021 of UniProt (human only).
- Query
  - Input
    - UniProt ID, Number of interacting human proteins
  - Output
    - The number of interacting human proteins from UniProt
    - If a UniProt ID is entered, it returns the number of interacting proteins
- Creators
  - 守屋

## Endpoint

https://rdfportal.org/sib/sparql

## `data`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?leaf ?label ?value
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  # union non-self-, self-, non-
  {
    SELECT ?leaf ?label (COUNT (DISTINCT ?target) AS ?value)
    WHERE {
      { # non-self-interaction
        VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
        ?leaf a up:Protein ;
              up:mnemonic ?label ;
              up:organism taxon:9606 ;
              up:proteome ?proteome ;
              up:interaction ?interaction .
        ?interaction a up:Non_Self_Interaction ;  # non self
                     ^up:interaction ?target .
        FILTER (?target != ?leaf)
      } UNION { # self-interaction
        VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
        ?leaf a up:Protein ;
              up:mnemonic ?label ;
              up:organism taxon:9606 ;
              up:proteome ?proteome ;
              up:interaction ?interaction .
        ?interaction a up:Self_Interaction ;  # self
                     ^up:interaction ?target .
        FILTER (?target = ?leaf)
      } 
    }
  } UNION { # non-interaction
    SELECT ?leaf ?label ?value
    WHERE {
      VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
      ?leaf a up:Protein ;
             up:mnemonic ?label ;
             up:organism taxon:9606 ;
             up:proteome ?proteome .
      MINUS { ?leaf up:interaction [] . }   
      BIND (0 AS ?value)
    }
  }
}
```

## `results`

```javascript
({data})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  
  return data.results.bindings.map(d => {
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
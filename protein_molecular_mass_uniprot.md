# UniProt molecular mass distribution

## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot/)
- Query
  - Input
    -  UniProt ID
  - Output
    - Molecular mass (kDa) range
- Creators
  - 守屋 

## Endpoint

https://rdfportal.org/sib/sparql

## `data`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?leaf ?label ?value        
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
  ?leaf a up:Protein ;
         up:mnemonic ?label ;
         up:organism taxon:9606 ;
         up:proteome ?proteome ;
         up:sequence/up:mass ?value .
}
```

## `return`
```javascript
({data})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  
  return data.results.bindings.map(d => {
    const num = parseInt(Number(d.value.value) / 10000);
    const bin_id = (num * 10) + "-" + ((num + 1) * 10);
    return {
      id: d.leaf.value.replace(idPrefix, ""),
      label: d.label.value,
      value: Number(d.value.value),
      binId: num + 1,
      binLabel: bin_id + " kDa"
    }
  })
}
```
# UniProt PDB-existence classification

## Description
 
- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot/)
- Query
  - Input
    - Existence (1: exists, 0 :not exists), uniprot id
  - Output
    - Uniprot entries are sorted by the presence or absence of a link to pdb.
    - If a uniprot id is entered, it returns whether pdb entry exists or not.
- Creator
   - 守屋
  
## Endpoint

https://rdfportal.org/sib/sparql

## `withAnnotation`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX db: <http://purl.uniprot.org/database/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?child ?child_label
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: } 
  ?child a up:Protein ;
         up:mnemonic ?child_label ;
         up:organism taxon:9606 ;
         up:proteome ?proteome ;
         rdfs:seeAlso/up:database db:PDB .
}
```

## `withoutAnnotation`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX db: <http://purl.uniprot.org/database/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?child ?child_label
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: } 
  ?child a up:Protein ;
         up:mnemonic ?child_label ;
         up:organism taxon:9606 ;
         up:proteome ?proteome .
  MINUS {
    ?child rdfs:seeAlso/up:database db:PDB . 
  }
}
```

## `return`
```javascript
({withAnnotation, withoutAnnotation})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const withoutId = "unclassified";
  
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    },{
      id: "1",
      label: "Proteins with structure data",
      parent: "root"
    },{
      id: withoutId,
      label: "Proteins without structure data",
      parent: "root"
    }
  ];  
  
  withAnnotation.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      parent: "1",
      leaf: true
    })
  })
  withoutAnnotation.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      parent: "unclassified",
      leaf: true
    })
  })
    
  return tree;
}
```
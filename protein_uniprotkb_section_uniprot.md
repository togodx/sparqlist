# UniProtKB section (SwissProt, TrEMBL) in the UniProt reference proteome

## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://www.uniprot.org/)
- Input/Output
  - Input
    - UniProt ID
  - Output
    - Reviewd flag (Swiss-Prot/TrEMBL)
- Creators
  - 守屋
## Endpoint

https://rdfportal.org/sib/sparql

## `withAnnotation`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
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
         up:proteome ?proteome ;
         up:reviewed true .
}
```

## `withoutAnnotation`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?leaf ?leaf_label
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
  ?leaf a up:Protein ;
        up:mnemonic ?leaf_label ;
        up:proteome ?proteome .
  MINUS { ?leaf up:reviewed true .}
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
      id: "reviewed",
      label: "Swiss-Prot",
      parent: "root"
    },{
      id: withoutId,
      label: "TrEMBL",
      parent: "root"
    }
  ];

  // アノテーション関係
  withAnnotation.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: "reviewed"
    })
  });
  // アノテーション無し要素
  withoutAnnotation.results.bindings.forEach(d => {
    tree.push({
      id: d.leaf.value.replace(idPrefix, ""),
      label: d.leaf_label.value,
      leaf: true,
      parent: withoutId
    });
  })
  
  return tree;
}
```
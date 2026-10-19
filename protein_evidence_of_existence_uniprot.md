# neXtProt protein-existence-level classification

## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot/)
- Query
  - Input
    - UniProt ID
  - Output
    - Protein existence obtained
- Creators
   - 守屋

## Endpoint 

https://rdfportal.org/sib/sparql

## `data`
```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?child ?parent ?child_label ?parent_label
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: } 
  ?child up:mnemonic ?child_label ;
         up:existence ?parent ;
         up:organism taxon:9606 ;
         up:proteome ?proteome .
  BIND (REPLACE (STRAFTER (STR (?parent), "/core/") , "_", " ") AS ?parent_label)
}
```

## `return`
```javascript
({data, allLeaf}) => {
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const withoutId = "without_annotation";
  const str2id = {
    "http://purl.uniprot.org/core/Evidence_at_Protein_Level_Existence": "1",
    "http://purl.uniprot.org/core/Evidence_at_Transcript_Level_Existence": "2",
    "http://purl.uniprot.org/core/Inferred_from_Homology_Existence": "3",
    "http://purl.uniprot.org/core/Predicted_Existence": "4",
    "http://purl.uniprot.org/core/Uncertain_Existence": "5"
  }
  
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  let edge = {};
  // アノテーション関係
  data.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: str2id[d.parent.value]
    })
    // root との親子関係を追加
    if (!edge[d.parent.value]) {
      edge[d.parent.value] = true;
      tree.push({     
        id: str2id[d.parent.value],
        label: d.parent_label.value,
        leaf: false,
        parent: "root"
      })
    }
  });
  
  return tree;
}
```
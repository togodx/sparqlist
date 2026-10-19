# UniProt isolated tissue classification

## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot/)
- Query
  - Obtain tissues to which input UniProt entries link with <https://www.uniprot.org/core/isolatedFrom>.
  - Input UniProt entries contain a reference describing the protein sequence obtained from a clone isolated from output tissues.
  - Input
    - UniProt ID
  - Output
    - Tissue
      - [UniProt Controlled vocabulary of tissues](https://www.uniprot.org/docs/tisslist)
- Creators
   - 守屋

## Endpoint

https://rdfportal.org/sib/sparql

## `data`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://sparql.uniprot.org/uniprot>
FROM <http://sparql.uniprot.org/tissues>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
  ?child a up:Protein ;
         up:mnemonic ?child_label ;
         up:organism taxon:9606 ;
         up:proteome ?proteome ;
         up:isolatedFrom ?parent .
  ?parent skos:prefLabel ?parent_label .
}
```

## `allLeaf`
- 全 UniProt (without annotation 用)
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
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
        up:organism taxon:9606 ;
        up:mnemonic ?leaf_label ;
        up:proteome ?proteome .
}
```

## `return`
```javascript
({data, allLeaf}) => {
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://purl.uniprot.org/tissues/";
  const withoutId = "unclassified"; 
  
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    },{
      id: withoutId,
      label: "Unclassified",
      parent: "root"
    }
  ];

  let withAnnotation = {};
  let edge = {};
  // アノテーション関係
  data.results.bindings.forEach(d => {
    withAnnotation[d.child.value] = true;
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    // root との親子関係を追加
    if (!edge[d.parent.value]) {
      edge[d.parent.value] = true;
      tree.push({     
        id: d.parent.value.replace(categoryPrefix, ""),
        label: d.parent_label.value,
        parent: "root"
      })
    }
  });
  // アノテーション無し要素
  allLeaf.results.bindings.forEach(d => {
    if (!withAnnotation[d.leaf.value]) {
      tree.push({
        id: d.leaf.value.replace(idPrefix, ""),
        label: d.leaf_label.value,
        leaf: true,
        parent: withoutId
      });
    }
  })
  
  return tree;
};
```
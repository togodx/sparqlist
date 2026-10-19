# UniProt ChEMBL-assay-confidence-score classification

- confidence score
  - https://chembl.gitbook.io/chembl-interface-documentation/frequently-asked-questions/chembl-data-questions

## Description
 
- Data sources
  - Proteins with or without ChEMBL assay from UniProt
  - ChEMBL-RDF: [https://rdfportal.org/dataset/chembl/](https://rdfportal.org/dataset/chembl/)
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot)
- Query
  - Input
    - Existence (1: exists, 0: not exists), UniProt ID
  - Output
    - The number of UniProt entries link to ChEMBL entries.
    - If a UniProt ID is entered, it returns whether ChEMBL entry exists or not.
- Creators
  - 守屋、信定

## Endpoint

https://rdfportal.org/ebi/sparql

## `data`
```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
SELECT DISTINCT ?child ?parent ?parent_label
FROM <http://rdf.ebi.ac.uk/dataset/chembl>
WHERE {
  ?chembl a cco:SmallMolecule ;
          cco:hasActivity/cco:hasAssay ?assay.
  ?assay a cco:Assay ;
            cco:targetConfScore ?parent ;
            cco:targetConfDesc ?parent_label ;
            cco:hasTarget/skos:exactMatch [
            cco:taxonomy taxon:9606 ;
            skos:exactMatch ?child
          ] . 
  ?child a cco:UniprotRef .
}
```

## Endpoint

https://rdfportal.org/sib/sparql

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
({data, allLeaf})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const withoutId = "unclassified";

  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    },{
      id: withoutId,
      label: "No ChEMBL assay",
      parent: "root"
    }
  ];

  let uri2label = {};
  allLeaf.results.bindings.forEach(d => {
    uri2label[d.leaf.value] = d.leaf_label.value;
  })

  let withAnnotation = {};
  let edge = {};
  // アノテーション関係
  data.results.bindings.forEach(d => {
    withAnnotation[d.child.value] = true;
    let parent_id;
    // id を conf-score [9-0] から sortable に
    if (d.parent.value.match(/^\d$/)) {
      parent_id = 10 - Number(d.parent.value);
      parent_id = parent_id.toString();
      d.parent_label.value = "Conf-score " + d.parent.value + ": " + d.parent_label.value;
    }
    if (uri2label[d.child.value]) { // uniprot referece proteome human にあるもの "UP000005640"
      tree.push({
        id: d.child.value.replace(idPrefix, ""),
        label: uri2label[d.child.value],
        leaf: true,
        parent: parent_id
      })
    }
    // root との親子関係を追加
    if (!edge[d.parent.value]) {
      edge[d.parent.value] = true;
      tree.push({     
        id: parent_id,
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
}
```
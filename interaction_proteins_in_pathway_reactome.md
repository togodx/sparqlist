# UniProt reactome-pathway classification

## Description

- Data sources
  - Reactome: [https://rdfportal.org/dataset/reactome/](https://rdfportal.org/dataset/reactome/)
    - The latest data can be obtained from the URL below. https://reactome.org/download-data
- Query
  - Input
    - Reactome ID, Uniprot ID
  - Output
    - The number of Uniprot entries included in each pathway in Reactome
    - If a Uniprot id is entered, it returns the pathway to which the protein belongs
- Creators
  - 守屋
  
## Endpoint

https://rdfportal.org/ebi/sparql

## `leaf`
- pathway と UniProt のアノテーション関係
- 産物と制御
  - 制御: reaction ^biopax:controlled/biopax:controller control-component
  - 産物: reaction biopax:left|biopax:right|biopax:product product-component
```sparql
PREFIX biopax: <http://www.biopax.org/release/biopax-level3.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?parent ?child ?parent_label
FROM <http://rdf.ebi.ac.uk/dataset/reactome>
WHERE {
  ?top_path a biopax:Pathway ;
            biopax:organism ?org .
  ?org biopax:name "Homo sapiens"^^xsd:string .
  MINUS { [] biopax:pathwayComponent ?top_path . }
  ?top_path biopax:pathwayComponent* ?path .
  ?path a biopax:Pathway ;
        biopax:displayName ?parent_label ;
        biopax:xref [
          biopax:db "Reactome"^^xsd:string ;
          biopax:id ?parent ;
        ] ;
        biopax:pathwayComponent ?reaction .
  ?reaction a biopax:BiochemicalReaction .
  ?reaction biopax:left|biopax:right|biopax:product|^biopax:controlled/biopax:controller ?component .
  ?component (biopax:memberPhysicalEntity|biopax:component)*/biopax:entityReference/biopax:xref [
    biopax:db "UniProt"^^xsd:string ;
    biopax:id ?child
  ] .
}
```

## `graph`
- pathway の親子関係
```sparql
PREFIX biopax: <http://www.biopax.org/release/biopax-level3.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://rdf.ebi.ac.uk/dataset/reactome>
WHERE {
  ?top_path a biopax:Pathway ;
            biopax:organism ?org .
  ?org biopax:name "Homo sapiens"^^xsd:string .
  MINUS { [] biopax:pathwayComponent ?top_path . }
  ?top_path biopax:pathwayComponent* ?parent_path .
  ?parent_path a biopax:Pathway ;
               biopax:displayName ?parent_label ;
               biopax:xref [
                 biopax:db "Reactome"^^xsd:string ;
                 biopax:id ?parent 
               ] ;
               biopax:pathwayComponent ?child_path .
  ?child_path a biopax:Pathway ;
              biopax:displayName ?child_label ;
              biopax:xref [
                biopax:db "Reactome"^^xsd:string ;
                biopax:id ?child 
              ] .
}
```

## `top`
- ヒト pathway の top 階層
```sparql
PREFIX biopax: <http://www.biopax.org/release/biopax-level3.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
SELECT DISTINCT ?top ?top_label
FROM <http://rdf.ebi.ac.uk/dataset/reactome>
WHERE {
  ?top_path a biopax:Pathway ;
            biopax:pathwayComponent ?child_path .
  MINUS { [] biopax:pathwayComponent ?top_path . }
  ?top_path biopax:organism ?org ;
            biopax:displayName ?top_label ;
            biopax:xref [
              biopax:db "Reactome"^^xsd:string ;
              biopax:id ?top ;
            ] .
  ?org biopax:name "Homo sapiens"^^xsd:string .
}
```

## Endpoint
https://rdfportal.org/sib/sparql

## `allLeaf`
- 全 UniProt (without annotation 用)
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX ch: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX chx: <http://purl.uniprot.org/proteomes/UP000005640#>
SELECT DISTINCT ?leaf ?leaf_label
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { ch:1 ch:2 ch:3 ch:4 ch:5 ch:6 ch:7 ch:8 ch:9 ch:10
                     ch:11 ch:12 ch:13 ch:14 ch:15 ch:16 ch:17 ch:18 ch:19 ch:20
                     ch:21 ch:22 ch:X ch:Y chx:Mitochondrion chx:Unplaced }
  ?leaf a up:Protein ;
        up:mnemonic ?leaf_label ;
        up:proteome ?proteome .
}
```

## `return`
- 整形
```javascript
({top, leaf, graph, allLeaf}) => {
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const withoutId = "unclassified";

  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    },{
      id: withoutId,
      label: "Not in the pathway",
      parent: "root"
    }
  ];
  
  let id2label = {};
  allLeaf.results.bindings.forEach(d => {
    id2label[d.leaf.value.replace(idPrefix, "")] = d.leaf_label.value;
  })

  let withAnnotation = {};
  // トップとルートの関係
  top.results.bindings.forEach(d => {
    tree.push({
      id: d.top.value.replace("R-HSA-", "R_HSA_"),
      label: d.top_label.value,
      parent: "root"
    })
  })
  // 親子関係
  graph.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value.replace("R-HSA-", "R_HSA_"),
      label: d.child_label.value,
      parent: d.parent.value.replace("R-HSA-", "R_HSA_")
    })
  })
  // アノテーション関係
  leaf.results.bindings.forEach(d => {
    withAnnotation[d.child.value] = true;
    if (id2label[d.child.value]) {  // proteome check
      tree.push({
        id: d.child.value,
        label: id2label[d.child.value],
        leaf: true,
        parent: d.parent.value.replace("R-HSA-", "R_HSA_")
      })
    }
  })
  // アノテーション無し要素
  allLeaf.results.bindings.forEach(d => {
    let id = d.leaf.value.replace(idPrefix, "");
    if (!withAnnotation[id]) {
      tree.push({
        id: id,
        label: d.leaf_label.value,
        leaf: true,
        parent: withoutId
      })
    }
  })
  
  return tree;
}
```
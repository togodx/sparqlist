# (TODO) Proteins specifically expressed in tissues (TiSMeD)

- Data sources
  - TiSMeD: [https://rdfportal.org/dataset/tismed/](https://rdfportal.org/dataset/tismed/)
- Input/Output
  - Input
    - UniProt ID
  - Output
    - TiSMeD tissue
- Creators
  - 池田・守屋
  
## Endpoint
https://rdfportal.org/primary/sparql

## `leaf`
```sparql
PREFIX tismedo: <http://med2rdf.org/tismed/ontology#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?child ?child_label ?parent
FROM <http://rdfportal.org/dataset/tismed>
WHERE {
  ?child tismedo:hasProteinExpressionSpecificity/tismedo:isPositivelySpecificTo ?parent .
  ?child rdfs:label ?child_label .
}
```

## `graph`
```sparql
PREFIX tismedo: <http://med2rdf.org/tismed/ontology#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

SELECT ?child ?child_label ?parent ?parent_label
FROM <http://rdfportal.org/dataset/tismed>
WHERE {
  ?child a owl:Class .
  ?child rdfs:subClassOf ?parent .
  ?child rdfs:label ?child_label .
  ?parent rdfs:label ?parent_label .
}
```

## Endpoint
https://rdfportal.org/sib/sparql

## `allLeaf`
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
        up:mnemonic ?leaf_label ;
        up:proteome ?proteome .
}
```

## `return`

```javascript
({leaf, graph, allLeaf}) => {
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://med2rdf.org/tismed/ontology#"
  const withoutId = "unclassified";
  
 let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    },{
      id: withoutId,
      label: "Low specificity",
      parent: "root"
    }
  ];

  let withAnnotation = {};
  let hasParent = {};
  // 親子関係
  graph.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value.replace(categoryPrefix, ""),
      label: d.child_label.value,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    hasParent[d.child.value] = true;
  })
  // root 直下
  graph.results.bindings.forEach(d => {
    if (!hasParent[d.parent.value]) {
      tree.push({
        id: d.parent.value.replace(categoryPrefix, ""),
        label: d.parent_label.value,
        parent: "root"
      })
      hasParent[d.parent.value] = true;
    }
  })
  // アノテーション関係
  leaf.results.bindings.forEach(d => {
    withAnnotation[d.child.value] = true;
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
  })
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
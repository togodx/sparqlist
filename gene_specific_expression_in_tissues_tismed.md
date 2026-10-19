# (TODO) Genes specifically expressed in tissues (TiSMeD)

- Data sources
  - TiSMeD: [https://rdfportal.org/dataset/tismed/](https://rdfportal.org/dataset/tismed/)
- Input/Output
  - Input
    - Ensembl gene ID
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
  ?child tismedo:hasGeneExpressionSpecificity/tismedo:isPositivelySpecificTo ?parent .
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
https://rdfportal.org/ebi/sparql

## `allLeaf`
```sparql
PREFIX so: <http://purl.obolibrary.org/obo/so#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>

SELECT DISTINCT ?leaf ?leaf_label
FROM <http://rdfportal.org/dataset/ensembl>
WHERE {
  [] so:transcribed_from ?leaf .
  ?leaf obo:RO_0002162 <http://ensembl.org/Homo_sapiens> ;
        rdfs:label ?leaf_label ;
        so:part_of ?chr .
  FILTER (REGEX (STR (?chr), "identifiers.org/hco/") && REGEX (STR (?chr), "GRCh38"))
}
```

## `return`

```javascript
({leaf, graph, allLeaf}) => {
  const idPrefix = "http://rdf.ebi.ac.uk/resource/ensembl/";
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
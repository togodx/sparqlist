# Diseases in Mondo

## Description

- Data sources
  -  [Mondo Disease Ontology (Mondo) ](https://mondo.monarchinitiative.org/) 
- Query
  - Input
    - Mondo id
  - Output
    -  [Disease and disorder (MONDO_0000001)](https://monarchinitiative.org/disease/MONDO:0000001) and its subcategories of Mondo
- Creators
  - 三橋、高月

## Endpoint

https://rdfportal.org/primary/sparql

## `data`
```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX mondo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

SELECT DISTINCT ?mondo ?label ?parent SAMPLE(?child) AS ?child
FROM <http://rdfportal.org/ontology/mondo>
WHERE {
  #root nodeはMONDO_0000001
  VALUES ?root { mondo:MONDO_0000001 }
  
  ?mondo rdfs:subClassOf+ ?root .
  ?mondo rdfs:label ?label.
  ?mondo rdfs:subClassOf ?parent.
  FILTER(!STRSTARTS(str(?parent), "nodeID:"))
  # ?parent rdfs:subClassOf* ?root . # root につながらない親を除外（帰ってこないので javascript で処理）
  # 中間ノードの場合は?parentに値が存在し、leafノードの場合は?parentは存在しないのでOPTIONALが必要
  OPTIONAL {
    ?mondo ^rdfs:subClassOf ?child.
  }
}
GROUP BY ?mondo ?parent ?label 
```
## `return`

```javascript
({data}) => {
  const idPrefix = "http://purl.obolibrary.org/obo/MONDO_";
  let tree = [
    {
      id: "0000001",
      label: "Disease and disorder",
      root: true
    }
  ];
  let mondo = { "0000001": true };
  data.results.bindings.forEach(d => {
    mondo[d.mondo.value.replace(idPrefix, "")] = true;
  });
  data.results.bindings.forEach(d => {
    if (mondo[d.parent.value.replace(idPrefix, "")]) { // root につながらない親を除外
      tree.push({
        id: d.mondo.value.replace(idPrefix, ""),
        label: d.label.value,
        leaf: (d.child == undefined ? true : false),
        parent: d.parent.value.replace(idPrefix, "")
      });
    }
  });
  return tree;
};
```

## MEMO
-Author
 - Takatsuki
 - Mitsuhashi
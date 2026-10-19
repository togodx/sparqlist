# Diseaseカテゴリフィルタ(HPO階層利用)

## Description

- Data sources
    -  [Human Phenotype Ontology (HPO)](https://hpo.jax.org/app/) 
- Query
  - Input
    - HPO ID
  - Output
    -  [Phenotypic abnormality (HP:0000118)](https://hpo.jax.org/app/browse/term/HP:0000118)  and its subcategories of HPO
- Creators
  - 高月、三橋

## Endpoint

https://rdfportal.org/primary/sparql

## `data`
```sparql

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

SELECT DISTINCT ?hp ?label ?parent (SAMPLE(?tmp_child) AS ?child)
FROM <http://rdfportal.org/ontology/hp>
WHERE {
  VALUES ?root {  obo:HP_0000118  }    #root nodeはHP_000118
  ?parent rdfs:subClassOf* ?root .
  ?hp a owl:Class ;
      rdfs:subClassOf ?parent ;
      rdfs:label ?label ;
      rdfs:subClassOf ?parent .
  FILTER regex(str(?hp), "http://purl.obolibrary.org/obo/HP_" )   # HP以外のIDも登録されているため、HPに限定するためにフィルターを追加
  # 中間ノードの場合は？parentに値が存在し、leafノードの場合は?parentは存在しないのでOPTIONALが必要
  OPTIONAL {
    ?tmp_child rdfs:subClassOf ?hp .
  }
} 
GROUP BY ?hp ?parent ?label
             
```
## `return`

```javascript
({data}) => {
  const idPrefix = "http://purl.obolibrary.org/obo/HP_";
  let tree = [
    {
      id: "0000118",
      label: "Phenotypic abnormality",
      root: true
    }
  ];
  data.results.bindings.forEach(d => {
    tree.push({
      id: d.hp.value.replace(idPrefix, ""),
      label: d.label.value,
      leaf: (d.child == undefined ? true : false),
      parent: d.parent.value.replace(idPrefix, "")
    });
  });
  return tree;
};
```

## MEMO
-Author
 - Takatsuki
 - Mitsuhashi
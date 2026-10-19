# Human phenotype ontorogy inheritance 分類

## Description
- HPのinheritanceのツリーに対して、OMIMのデータをMONDOへ変換をしてリーフにする
- Data sources
  - [Human Phenotype Ontology (HPO)](https://hpo.jax.org/app/) 
- Query
  - Input
    - HPO ID
  - Output
    -  [Mode of inheritance (HP:0000005)](http://purl.obolibrary.org/obo/HP_0000005)  and its subcategories of HPO
- Creators
  - 高月

## Endpoint

https://rdfportal.org/primary/sparql

## `leaf`
```sparql
PREFIX oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tio: <http://togoid.dbcls.jp/ontology#>

SELECT DISTINCT ?mondo ?mondo_id ?mondo_label ?hp
FROM <http://rdfportal.org/ontology/mondo>
FROM <http://rdfportal.org/dataset/togoid/relation/mondo-omim_phenotype>
FROM <http://rdfportal.org/dataset/togoid/relation/hp_inheritance-omim_phenotype>
WHERE{
  ?mondo rdfs:label ?mondo_label ;
         oboInOwl:id ?mondo_id . 
  ?mondo tio:TIO_000001 / tio:TIO_000104 ?hp .
}
```

## `graph`
```sparql

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

SELECT DISTINCT ?hp ?label ?parent (SAMPLE(?child) AS ?child)
FROM <http://rdfportal.org/ontology/hp>
WHERE {
  #root nodeはHP:0000005
  VALUES ?root {  obo:HP_0000005  }    
  ?hp rdfs:subClassOf+ ?root .
  ?hp rdfs:label ?label .
  ?hp rdfs:subClassOf ?parent .
  ?hp rdf:type owl:Class .  
  # HP以外のIDも登録されているため、HPに限定するためにフィルターを追加
  FILTER regex(str(?hp), "http://purl.obolibrary.org/obo/HP_" )
  # 中間ノードの場合は？parentに値が存在し、leafノードの場合は?parentは存在しないのでOPTIONALが必要
  OPTIONAL {
    ?hp ^rdfs:subClassOf ?child .
  }
 } 
 GROUP BY ?hp ?parent ?label

```
## `return`

```javascript
({graph,leaf}) => {
  const idPrefix = "http://purl.obolibrary.org/obo/";
  const leafPrefix= "http://purl.obolibrary.org/obo/MONDO_";
  let tree = [
    {
      id: "HP_0000005",
      label: "Mode of inheritance",
      root: true
    }
  ];
  graph.results.bindings.forEach(d => {
    tree.push({
      id: d.hp.value.replace(idPrefix, ""),
      label: d.label.value,
      parent: d.parent.value.replace(idPrefix, "")
    })
  })
  leaf.results.bindings.forEach(d => {
    tree.push({
      id: d.mondo.value.replace(leafPrefix, ""),
      label: d.mondo_label.value,
      parent: d.hp.value.replace(idPrefix, ""),
      leaf: true
    });
  });
  return tree;
};
```

## MEMO
-Author
 - Takatsuki
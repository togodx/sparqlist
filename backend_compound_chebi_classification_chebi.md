# ChEBI classification での分類 

- backend SPARQLet
  - ChEBI: application type, biological role, chemical role
- アノテーションがないケースを数えていない（もとのまま）
- Creators
  - 川島、建石、信定

## Parameters

* `root`
  * default: 33232
  * example: 33232 (Application type), 24432 (Biological role), 51086 (Chemical role)

## Endpoint

https://rdfportal.org/ebi/sparql

## `graph`
- Application の親子関係

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://rdf.ebi.ac.uk/dataset/chebi>
WHERE 
{
      
  ?r a owl:Restriction ;
    owl:onProperty obo:RO_0000087 ;
    owl:someValuesFrom ?chebi_class .
  ?parent rdfs:subClassOf* obo:CHEBI_{{root}} .
  ?child rdfs:subClassOf ?parent .
  ?child rdfs:label ?child_label .
  ?parent rdfs:label ?parent_label .
  ?chebi_class rdfs:subClassOf* ?child .
  
}
```

## `leaf`
- ChEBI Compound  のアノテーション

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT distinct ?compound ?category (GROUP_CONCAT(DISTINCT ?label; SEPARATOR = ", ") as ?compound_label)                                
FROM <http://rdf.ebi.ac.uk/dataset/chebi>
WHERE 
{     
  ?compound a owl:Class ;
    rdfs:label ?label ;
    rdfs:subClassOf ?r .
  ?r a owl:Restriction ;
    owl:onProperty obo:RO_0000087 ;
    owl:someValuesFrom ?category .
  ?category rdfs:subClassOf*  obo:CHEBI_{{root}} .
}
```

## `return`

```javascript
({root, leaf, graph}) => {
  const idPrefix = "http://purl.obolibrary.org/obo/CHEBI_";
  const categoryPrefix = "http://purl.obolibrary.org/obo/CHEBI_";
  
  let tree = [
    {
      id: root,
      root: true
    }
  ];

  // 親子関係
  graph.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value.replace(categoryPrefix, ""),
      label: d.child_label.value,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label) tree[0].label = d.parent_label.value; // root の label 挿入
  })
  // アノテーション関係
  leaf.results.bindings.forEach(d => {
    tree.push({
      id: d.compound.value.replace(idPrefix, ""),
      label: d.compound_label.value,
      leaf: true,
      parent: d.category.value.replace(categoryPrefix, "")
    })
  })
  
  return tree;	
}
```
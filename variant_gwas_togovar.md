# variant GWAS (Genome-wide association study) 

## Description

- Data sources
  -  [TogoVar](https://togovar.biosciencedbc.jp/?) (limited to variants with frequency data in Japanese populations)
- Query
  - Input
    - TogoVar ID
  - Output
    -  [Mapped trait used in GWAS Catalog](https://www.ebi.ac.uk/gwas/docs/ontology)
- Creators
  - 三橋、守屋

## Endpoint

https://grch38.togovar.org/sparql

## `leaf`
```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX efo: <http://www.ebi.ac.uk/efo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX terms: <http://med2rdf.org/gwascatalog/terms/>

SELECT DISTINCT ?efo AS ?parent ?efo_label AS ?parent_label ?tgv_id AS ?child  ?rs_uri AS ?child_label
FROM <http://togovar.org/efo>
FROM <http://togovar.org/gwas-catalog>
FROM <http://togovar.org/variant>
FROM <http://togovar.org/variant/annotation/ensembl>
WHERE {
  VALUES ?root {  efo:EFO_0000001  } 
  GRAPH <http://togovar.org/efo>{
    ?efo rdfs:label ?efo_label ;
      rdfs:subClassOf* ?root;  # The parent must be reachable from the root.
      a owl:Class .
  }
  GRAPH <http://togovar.org/gwas-catalog>{
    ?efo ^terms:mapped_trait_uri ?variant_gwas_catalog.
    ?variant_gwas_catalog rdfs:seeAlso ?rs_uri .
  }
  GRAPH <http://togovar.org/variant/annotation/ensembl> {
    ?rs_uri  ^rdfs:seeAlso ?variant_togovar .
  }
  GRAPH <http://togovar.org/variant> {
    ?variant_togovar dct:identifier ?tgv_id .
  }
}
```

## `graph`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX terms: <http://med2rdf.org/gwascatalog/terms/>
PREFIX efo: <http://www.ebi.ac.uk/efo/>
SELECT DISTINCT ?parent ?child ?child_label
FROM <http://togovar.org/efo>
FROM <http://togovar.org/gwas-catalog>
FROM <http://togovar.org/variant/annotation/ensembl>
WHERE {
  VALUES ?root {  efo:EFO_0000001  } 
  GRAPH <http://togovar.org/efo> {
    ?child a owl:Class ;
      rdfs:subClassOf ?parent ;
      rdfs:label ?child_label .
    ?parent a owl:Class ;
      rdfs:subClassOf* ?root .
    ?efo rdfs:subClassOf* ?child .
  }
  # Ensure that an EFO entry links to a TogoVar variant via GWAS Catalog
  GRAPH <http://togovar.org/gwas-catalog>{
    ?efo ^terms:mapped_trait_uri ?variant_gwas_catalog.
    ?variant_gwas_catalog rdfs:seeAlso ?rs_uri .
  }
  GRAPH <http://togovar.org/variant/annotation/ensembl> {
    ?rs_uri  ^rdfs:seeAlso ?variant_togovar .
  }
} 
```

## `return`
```javascript
({leaf, graph}) => {
  const childLabelPrefix = "http://identifiers.org/dbsnp/";
  
  let tree = [
    {
      id: "EFO_0000001",
      label: "experimental factor",
      root: true
    }
  ];

  let is_dumped = {}  // 既に出力されたid,parent関係
 
  // 親子関係
  graph.results.bindings.forEach(d => {
    id = d.child.value.split(/\//).slice(-1)[0]
    parent = d.parent.value.split(/\//).slice(-1)[0]
    if(is_dumped[id] == parent){ return }   // 一度出力されたid,parentは出力しない
    is_dumped[id] = parent
    tree.push({
      id: id,
      label: d.child_label.value,
      parent: d.parent.value.split(/\//).slice(-1)[0]
    })
  })
  // アノテーション関係
  leaf.results.bindings.forEach(d => {
    parent = d.parent.value.split(/\//).slice(-1)[0]
    tree.push({
      id: d.child.value,
      label: d.child_label.value.replace(childLabelPrefix, ""),
      leaf: true,
      parent: d.parent.value.split(/\//).slice(-1)[0]
    })
  })
  
  return tree;	
}
```
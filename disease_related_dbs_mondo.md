# MONDOにおける、関連付けされているDBのカウント(高月・三橋)

## Description

- Data sources
  - Mondo disease ontlogy: [https://mondo.monarchinitiative.org/](https://mondo.monarchinitiative.org/)
- Query
  - Input
    - MONDO ID
  - Output
    - Related Databases category
- Supplementary information
  - The data counted the number of associations for each of the approximate 70 disease DBs associated with the lexical data MONDO, which is an exhaustive collection of disease names.
  - 網羅的に疾患名を集めている語彙データMONDOに関連づけられているおおよそ70の各疾患DBに対して、関連付けされた数をカウントしたデータ。
- Creators
  - 高月、三橋

## Endpoint

https://rdfportal.org/primary/sparql

## `data`

```sparql
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>


SELECT DISTINCT ?mondo ?label ?parent
FROM <http://rdfportal.org/ontology/mondo>
WHERE {
  ?mondo rdfs:label ?label.
  ?mondo oboinowl:hasDbXref ?id .
  BIND (strbefore(str(?id), ":") AS ?parent)  
  FILTER(!STRSTARTS(str(?id), "http"))
  FILTER regex(str(?mondo), "http://purl.obolibrary.org/obo/MONDO_" )
 VALUES ?parent {"UMLS" "ICD10" "DOID" "Orphanet" "OMIM" "SCTID" "MESH"
                 "NCIT" "GARD"  "ICD9" "EFO" "COHD" "MedDRA" "GC_ID" "SNOMEDCT_US"
                 "PMID" "ICD0" "HP" "ONCOTREE" "OMIMPS" "MSH" "FMA" "DC" "BTO" "CALOHA"}
}

```
## `return`

```javascript
({data}) => {
  const idPrefix = "http://purl.obolibrary.org/obo/MONDO_";
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  let edge = {};
  data.results.bindings.forEach(d => {
    tree.push({
      id: d.mondo.value.replace(idPrefix, ""),
      label: d.label.value,
      leaf: true,
      parent: d.parent.value
    })
  // root との親子関係を追加
    if (!edge[d.parent.value]) {
      edge[d.parent.value] = true;
      tree.push({   
        id: d.parent.value,
        label: d.parent.value,
        leaf: false,
        parent: "root"
      })
    }
  });
  
  return tree;
};
```


## MEMO
-Author
 - Takatsuki
 - Mitsuhashi
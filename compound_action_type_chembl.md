# ChEMBLを作用機作で分類する
server対応済み

## Description

- Data sources
  - ChEMBL: [https://rdfportal.org/dataset/chembl/](https://rdfportal.org/dataset/chembl/)
- Query
  -  Input
    - ChEMBL ID
  - Output
    - Mechanism action type
- Creators
  - 信定、八塚、鈴木

## Endpoint

https://rdfportal.org/ebi/sparql

## `data`

```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT?parent ?child ?child_label  
FROM <http://rdf.ebi.ac.uk/dataset/chembl>
WHERE 
{
  ?chembl_drug_mechanism a cco:Mechanism ;
                        cco:hasMolecule ?molecule ;
                        cco:mechanismActionType ?parent .
 ?molecule cco:chemblId ?child ;
           rdfs:label ?child_label .
}

```

## `return`

```javascript
({data}) => {
  
  let tree = [
    {
      id: "root",
      // 2023-05-31 tateisi
      // label: "root node",
      label: "action type",
      root: true
    }
  ];

  let edge = {};
  data.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value,
      label: d.child_label.value,
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
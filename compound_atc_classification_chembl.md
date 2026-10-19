# ChEMBL で薬を薬効（WHO ATC Code）で分類 (Server 対応だが、アノテーションのないものを数えていない）

* 入力：
  * 
* 出力：
  * ChEMBL の化合物でWHO ATC コードの付与されている化合物をATCコードとともに出力
  * 一つの薬に複数のATCコードがついていることがありうる 

## Description

- Data sources
  - ATC: https://bioportal.bioontology.org/ontologies/ATC
  - ChEMBL: [https://rdfportal.org/dataset/chembl/](https://rdfportal.org/dataset/chembl/)
- Query
  - Input
    - ChEMBL ID
  - Output
    - ATC category ID
    - ATC category label
- Creators
  - 建石、山本

## Endpoint

https://rdfportal.org/ebi/sparql

## `data`
- 化合物とATCコードの対応
- ソートしていない

```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#> 
PREFIX molecule: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?atc ?molecule ?molecule_label
FROM <http://rdf.ebi.ac.uk/dataset/chembl>

WHERE {
  # test
  #VALUES ?molecule {  
  #  molecule:CHEMBL17860  molecule:CHEMBL231779  molecule:CHEMBL231813  molecule:CHEMBL251634  molecule:CHEMBL292707
  #}
  
  ?molecule cco:atcClassification ?atc;
            rdfs:label ?molecule_label.
        
}
```


## `atcGraph` 
-  ATCコードの階層

```javascript
({data})=>{
  let parents={}
  data.results.bindings.forEach(d=>{
    let atc=d.atc.value;
    if (atc.length==7) {
      let child=atc;
      atc=atc.substr(0,5)
      parents[child]=atc 
    }
    if (atc.length==5) {
      let child=atc;
      atc=atc.substr(0,4)
      parents[child]=atc 
    }
    if (atc.length==4) {
      let child=atc;
      atc=atc.substr(0,3)
      parents[child]=atc 
    }
    if (atc.length==3) {
      let child=atc;
      atc=atc.substr(0,1)
      parents[child]=atc 
      
    }
    if (atc.length==1) {
      let child=atc;
      atc=atc.substr(0,1)
      parents[child]="root" 
    }
  });	
  return(parents)
}
```

## `atcArray`
- ATCコードのリスト

```javascript
({atcGraph})=>{
  return (Object.keys(atcGraph))
}
```

## Endpoint

https://rdfportal.org/bioportal/sparql

## `labelData`
* ATCコードのラベル取得
* Endpointが https://integbio.jp/rdf/mirror/bioportal/sparql のとき
  FROM <http://integbio.jp/rdf/mirror/bioportal/atc>

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?atc ?label 
FROM <http://rdfportal.org/dataset/bioportal/atc>
WHERE 
{
    
    ?atcuri  skos:prefLabel ?label.  
    # BIND(substr(str(?atcuri),42) as ?atc)  
    # BIND(substr(str(?atcuri),43) as ?atc) 
    BIND(replace(str(?atcuri),".*/([A-Z0-9])","$1") as ?atc) 
}
```

## `return`

```javascript
({data, atcGraph, atcArray, labelData}) => {
  const idPrefix = "http://rdf.ebi.ac.uk/resource/chembl/molecule/";
  const withoutId = "unclassified";
  
  let tree = [
    {
      id: "root",
      // label: "root node",
      label: "ATC",
      root: true
    },
    {
      id: withoutId,
      label: "without annotation",
      parent: "root"
    }
  ];

  let atcLabel={}
  labelData.results.bindings.forEach(d=>{
    atcLabel[d.atc.value]=d.label.value
  });
   
  // アノテーション
  data.results.bindings.forEach(d => {
    
    tree.push({
      id: d.molecule.value.replace(idPrefix, ""),
      label: d.molecule_label.value,
      leaf: true,
      parent: d.atc.value
    })
  })
  
  // ATCコードの階層
  atcArray.forEach(d=>{
    tree.push({     
        id: d,
        label: (atcLabel[d] == undefined ? "undef (ATC alteration)" : atcLabel[d]),
        leaf: false,
        parent: atcGraph[d]
      })
  })
  return tree;
};
```
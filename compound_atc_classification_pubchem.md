# PubChem で薬を薬効で分類（FDA Approved Drugs → WHO ATC Code）
- (Annotationのない化合物を数えていない＝もとのまま)

## Description

- Data sources
  - PubChem: 
    - Data for nodes linked to ChEMBL or ChEBI retrieved from https://integbio.jp/rdf/dataset/pubchem
- Query
  - Input
    - PubChem Compound ID 
  - Output
    - WHO ATC code (https://www.whocc.no/atc_ddd_index/)
- Creators
  - 建石、守屋、山本

## Endpoint

https://rdfportal.org/pubchem/sparql

## Parameters

## `data`

```sparql
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT DISTINCT ?cid ?pubchem_label ?atc
WHERE {
 ?attr a sio:CHEMINF_000562 ;
       sio:SIO_000011 ?cid ;
       sio:SIO_000300 ?inn ;
       dcterms:subject ?atc .
  # ?atc a skos:concept .  # EP にないので FILTER で
  FILTER (REGEX (STR (?atc), "concept/ATC_"))
  ?cid sio:SIO_000008  [ 
    a sio:CHEMINF_000382 ;
    sio:SIO_000300 ?pubchem_label_temp  
  ] .

  BIND(IF(bound(?pubchem_label_temp), ?pubchem_label_temp, "null") AS ?pubchem_label)   
} 
```

## Endpoint

https://rdfportal.org/bioportal/sparql

## `atcGraph`

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT  ?atc ?atc_label ?parent ?parent_label
FROM <http://rdfportal.org/dataset/bioportal/atc>
WHERE {
  ?atc rdfs:subClassOf* ?atctop ;
       skos:prefLabel  ?atc_label .
  # ?atc  a skos:concept .  # EP にないので FILTER で
  FILTER (REGEX (STR (?atc), "ontology/ATC/"))
  OPTIONAL {
    ?atc rdfs:subClassOf ?parent .      
    ?parent skos:prefLabel  ?parent_label .
    FILTER (?atc != ?parent)
  }
}
ORDER BY ?atc
```

## `return`
- 整形

```javascript
({data,atcGraph})=>{
  const idVarName = "cid";
  const idPrefix = "http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID";
  const idLabelName="pubchem_label";
  const categoryVarName = "atc";
  const categoryLabelVarName = "atc_label";
  const categoryPrefix = "http://rdf.ncbi.nlm.nih.gov/pubchem/concept/ATC_";
  const categoryPrefix2 = "http://purl.bioontology.org/ontology/ATC/";
  const withoutId = "unclassified";
  
  function capitalize(str) {
    return str.charAt(0).toUpperCase() + str.substring(1);
  }
  
  let tree = [
    {
      id: "root",
      // 2023-05-31 tateisi
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
  
  let atcLabel={};
  atcGraph.results.bindings.forEach(d=>{
    atcLabel[d.atc.value.replace(categoryPrefix2, "")]=d.atc_label.value;
  });
   
  // Annotation
  data.results.bindings.forEach(d=>{
    if (!atcLabel[d.atc.value.replace(categoryPrefix, "")]) return 0; // ディスコン ATC ID
    tree.push({
      id: d[idVarName].value.replace(idPrefix, ""), 
      label: d[idLabelName].value,
      leaf: true,
      parent: d[categoryVarName].value.replace(categoryPrefix, ""), 
    })
  })
  
  //Category tree
  atcGraph.results.bindings.forEach(d=>{
    let child=d[categoryVarName].value.replace(categoryPrefix2, "")
    if (child.length==1){
      tree.push({     
        id: child,
        label: d.atc_label.value,
        leaf: false,
        parent: "root",
      })
    } else {    
      tree.push({     
        id: child,
        label: d.atc_label.value,
        leaf: false,
        parent: d.parent.value.replace(categoryPrefix2, "")
      })
    }  
  })
  
  
  
  return tree ;
}
```
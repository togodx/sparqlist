# Genes on chromosomes

## Description

- Data sources
    -     - Ensembl human: [https://rdfportal.org/dataset/ensembl/](https://rdfportal.org/dataset/ensembl/)
- Input/Output
    -  Input
        - Ensembl gene ID
    - Output
        - Chromosome number
- Supplementary information
    - The chromosome on which each human gene is located.
    - ヒトの各遺伝子が位置している染色体の番号を示します。
- Creators
  - 片山、池田、守屋

## Endpoint

https://rdfportal.org/ebi/sparql

## `data`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX so: <http://purl.obolibrary.org/obo/so#>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dcterms: <http://purl.org/dc/terms/>

SELECT DISTINCT ?parent ?child ?child_label
FROM <http://rdfportal.org/dataset/ensembl>
WHERE {
  ?enst so:transcribed_from ?ensg .
  ?ensg obo:RO_0002162 <http://ensembl.org/Homo_sapiens> ;
        dcterms:identifier ?child ;
        rdfs:label ?child_label ;
        so:part_of ?chr .
  FILTER (REGEX (STR (?chr), "identifiers.org/hco/") && REGEX (STR (?chr), "GRCh38"))
  BIND(STRBEFORE(STRAFTER(STR(?chr), "hco/"), "/GRCh38") AS ?parent)
 # VALUES ?parent {
 #     "1" "2" "3" "4" "5" "6" "7" "8" "9" "10"
 #     "11" "12" "13" "14" "15" "16" "17" "18" "19" "20" "21" "22"
 #     "X" "Y" "MT"
 # }
}
```

## `return`

```javascript
({data}) => {
  const idPrefix = "http://rdf.ebi.ac.uk/resource/ensembl/";
  
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  let edge = {};
  // アノテーション関係
  data.results.bindings.forEach(d => {
    // 分類ノード id を sortable に
    let parent_id = d.parent.value;
    if (parent_id == "X") parent_id = "23";
    else if (parent_id == "Y") parent_id = "24";
    else if (parent_id == "MT") parent_id = "25";
    else {
      parent_id = ('00' + parent_id).slice(-2);
      d.parent.value = "chr" + d.parent.value;
    }
    let label = d.child_label.value;
    if (label == "") {
      label = d.child.value;
    }
    tree.push({
      id: d.child.value,
      label: label,
      leaf: true,
      parent: parent_id
    })
    // root との親子関係を追加
    if (!edge[d.parent.value]) {
      edge[d.parent.value] = true;
      tree.push({     
        id: parent_id,
        label: d.parent.value,
        leaf: false,
        parent: "root"
      })
    }
  });
  
  return tree;
};
```
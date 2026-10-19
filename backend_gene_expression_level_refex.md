# RefEx organ specific expression based on ROKU flag

* backend SPARQLet
* Creators
  * 守屋、池田

## Parameters

* `negatively` (低発現)
  * example: 1

## Endpoint
https://rdfportal.org/primary/sparql

## `data`
```sparql
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://purl.jp/bio/01/refexo>
FROM <http://refex.dbcls.jp/rdf/tissue>
FROM <http://rdfportal.org/dataset/homologene>
WHERE {
  ?child refexo:affyProbeset ?affy ;
         rdfs:label ?child_label .
{{#if negatively}}
  ?affy refexo:isNegativelySpecificTo ?parent .
{{else}}
  ?affy refexo:isPositivelySpecificTo ?parent .
{{/if}}
  ?parent rdfs:label ?parent_label .
  FILTER (LANG (?parent_label) = "en")
}
```

## `allLeaf`
- 全 NCBI gene (without annotation 用)
```sparql
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?leaf ?leaf_label
FROM <http://refex.dbcls.jp/rdf/tissue>
FROM <http://rdfportal.org/dataset/homologene>
WHERE {
  ?leaf refexo:affyProbeset [] ;
        rdfs:label ?leaf_label .
}
```

## `return`

```javascript
({data, allLeaf})=>{
  const idPrefix = "http://identifiers.org/ncbigene/";
  const categoryPrefix = "http://purl.jp/bio/01/refexo#";
  const withoutId = "unclassified";

  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    },{
      id: withoutId,
      label: "Low specificity",
      parent: "root"
    }
  ];

  let withAnnotation = {};
  let edge = {};
  // アノテーション関係
  data.results.bindings.forEach(d => {
    withAnnotation[d.child.value] = true;
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    // root との親子関係を追加
    if (!edge[d.parent.value]) {
      edge[d.parent.value] = true;
      tree.push({
        id: d.parent.value.replace(categoryPrefix, ""),
        label: d.parent_label.value,
        parent: "root"
      })
    }
  }) 
  // アノテーション無し要素
  allLeaf.results.bindings.forEach(d => {
    if (!withAnnotation[d.leaf.value]) {
      tree.push({
        id: d.leaf.value.replace(idPrefix, ""),
        label: d.leaf_label.value,
        leaf: true,
        parent: withoutId
      });
    }
  })

  return tree;
}
```
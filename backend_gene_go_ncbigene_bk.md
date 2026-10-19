# NCBI Gene GO classification

- backend SPARQLet
- Creators
  - 池田

## Parameters

* `root` (type: GO) (Req.)
  * default: GO_0005575
  * example: GO_0008150 (biological process), GO_0005575 (cellular component), GO_0003674 (molecular function)

## Endpoint

https://rdfportal.org/primary/sparql

## `leaf`
- GO と NCBI Gene のアノテーション関係
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tio: <http://togoid.dbcls.jp/ontology#>
SELECT DISTINCT ?parent ?child ?child_label
FROM <http://rdfportal.org/dataset/togoid/relation/ncbigene-go>
FROM <http://rdfportal.org/dataset/togoid/relation/ncbigene-hgnc>
FROM <http://rdfportal.org/dataset/togoid/relation/hgnc-hgnc_symbol>
FROM <http://rdfportal.org/ontology/go>
WHERE {
  ?child tio:TIO_000004 ?parent ;
         tio:TIO_000001 / tio:TIO_000022 ?child_tmp_label .
  ?parent rdfs:subClassOf* obo:{{root}} .
  BIND (STRAFTER (STR (?child_tmp_label), "symbol/") AS ?child_label)
}
```

## `graph`
- GO の親子関係
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?parent ?child ?child_label ?parent_label
FROM <http://rdfportal.org/ontology/go>
WHERE {
  ?child rdfs:subClassOf* obo:{{root}} ;
  #?child rdfs:subClassOf* obo:GO_0008150 ;
         rdfs:subClassOf ?parent .
  FILTER(REGEX(STR(?child), "GO_"))
  FILTER(REGEX(STR(?parent), "GO_"))
  ?child rdfs:label ?child_label .
  ?parent rdfs:label ?parent_label .
}
```

## `allLeaf`
- 全 NCBI Gene (without annotation 用)
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tio: <http://togoid.dbcls.jp/ontology#>
SELECT DISTINCT ?leaf ?leaf_label
FROM <http://rdfportal.org/dataset/togoid/relation/ncbigene-hgnc>
FROM <http://rdfportal.org/dataset/togoid/relation/hgnc-hgnc_symbol>
WHERE {
  ?leaf tio:TIO_000001 / tio:TIO_000022 ?leaf_tmp_label .
  BIND (STRAFTER (STR (?leaf_tmp_label), "symbol/") AS ?leaf_label)
}
```

## `return`
```javascript
({root, leaf, graph, allLeaf}) => {
  const idPrefix = "http://identifiers.org/ncbigene/";
  const categoryPrefix = "http://purl.obolibrary.org/obo/";
  const withoutId = "unclassified";
  
  let tree = [
    {
      id: root,
      root: true
    },{
      id: withoutId,
      label: "Unclassified",
      parent: root
    }
  ];

  let withAnnotation = {};
  // 親子関係
  graph.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value.replace(categoryPrefix, ""),
      label: d.child_label.value,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label)
      tree[0].label = d.parent_label.value; // root の label 挿入
  })
  // アノテーション関係
  leaf.results.bindings.forEach(d => {
    //leaf が root を直接指している場合、evidence code が ND (No biological Data available)
    //unclassfied と区別したほうがよいかもしれないが、とりあえず同じ扱いにしておく
    if (d.parent.value.replace(categoryPrefix, "") == root)
      return;
    withAnnotation[d.child.value] = true;
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
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
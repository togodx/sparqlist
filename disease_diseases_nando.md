# 日本の難病疾患のカテゴリフィルター（NANDO階層利用）

## Description

- Data sources
  - Nanbyo Disease Ontology (NANDO): [https://rdfportal.org/dataset/nando/](https://rdfportal.org/dataset/nando/)
- Query
  - Input
    - NANDO ID
  - Outpu
    - NANDO category
- Supplementary information
- Creators
  - 高月、三橋

## Endpoint

https://rdfportal.org/primary/sparql

## `data`

```sparql
PREFIX nando: <http://nanbyodata.jp/ontology/NANDO_>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?id ?parent ?label SAMPLE(?child) AS ?child
FROM <http://nanbyodata.jp/ontology/nando>
WHERE {
  # ルートノードは、NANDO:0000001（Intractable disease）
  VALUES ?root { nando:0000001 }
  
  # ルートノードのJSONはreturn(次のステップのJavaScript)で作成するのでsubClassOf*ではなくsubClassOf+とする。
  ?id rdfs:subClassOf+ ?root.
  ?id rdfs:label ?label.
  FILTER (lang(?label) = "en")
  ?id rdfs:subClassOf ?parent.
  
  # 中間ノードの場合は?childに値が存在し、leafノードの場合は?childは存在しないのでOPTIONALが必要
  OPTIONAL {
    ?id ^rdfs:subClassOf ?child.
  }
}
GROUP BY ?id ?parent ?label 
```

## `return`

```javascript
({data}) => {
  const idPrefix = "http://nanbyodata.jp/ontology/NANDO_";
  let tree = [
    {
      id: "0000001",
      label: "Intractable disease",
      root: true
    }
  ];
  data.results.bindings.forEach(d => {
    tree.push({
      id: d.id.value.replace(idPrefix, ""),
      label: d.label.value,
      leaf: (d.child == undefined ? true : false),
      parent: d.parent.value.replace(idPrefix, "")
    });
  });
  return tree;
};
```

## MEMO
-Author
 - Takatsuki
 - Mitsuhashi
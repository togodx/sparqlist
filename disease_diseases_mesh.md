# Diseases in MeSH

* MeSH の D番号を node ID として使うと、情報が減って DAG がループしてしまうので、Tree番号をIDとする
  * 中間ノードへのIDのマッピングのために、すべての tree ID node に D番号を leaf としてぶら下げる
  * attributes.yml の ontology フラグによる、nested_set_indexer の --conplete-leaf オプションでの leaf の補間的な作業を SPARQList 側でしておく

## Description

- Data sources
  -  Medical Subject Headings (MeSH): [https://rdfportal.org/dataset/mesh/](https://rdfportal.org/dataset/mesh/)
- Creators
  - 三橋、守屋

## Endpoint

https://rdfportal.org/ncbi/sparql

## testパターン
- この例は、[0_readme](https://togodx.integbio.jp/sparqlist_dev/0_readme)の「分類系：ルートノードが無い場合や、階層が無い場合」と近く、「分類系：ルートノードが無いが、階層がある場合」。
- parentがrootの例 ([Infections: D007239](https://meshb.nlm.nih.gov/record/ui?ui=D007239))
  - returnのJSONを「"id": "D007239"」で検索する
 ```
  {
    "id": "D007239",
    "label": "Infections",
    "leaf": false,
    "parent": "root"
  },
 ```
- parentが複数(DAG)かつleafの例 ([Aneurysm, Infected: D000785](https://meshb.nlm.nih.gov/record/ui?ui=D000785))
   -  returnのJSONを「"id": "D000785"」で検索する
```
  {
    "id": "D000785",
    "label": "Aneurysm, Infected",
    "leaf": true,
    "parent": "D007239"
  },
  {
    "id": "D000785",
    "label": "Aneurysm, Infected",
    "leaf": true,
    "parent": "D000783"
  },
```

## `data`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>

SELECT DISTINCT ?mesh_id ?mesh_label ?parent_mesh_id ?node ?parent_node
FROM <http://id.nlm.nih.gov/mesh>
WHERE {
  # MeSH Treeの Diseases[C] 以下を取得
  # See https://meshb.nlm.nih.gov/treeView
  FILTER (contains(str(?node), "C"))
  OPTIONAL {
    ?node meshv:parentTreeNumber ?parent_node  .
    ?parent_mesh_id meshv:treeNumber ?parent_node .
  }

  ?mesh_id meshv:treeNumber ?node ;
    rdfs:label ?mesh_label .
  FILTER(lang(?mesh_label) = "en")
}
```

## `return`

```javascript
({data}) => {
  const idPrefix = "http://id.nlm.nih.gov/mesh/";
  let tree = [
    {
      id: "root",
      label: "Diseases",
      root: true
    }
  ];
  data.results.bindings.forEach(d => {
    tree.push({
      id: d.node.value.replace(idPrefix, ""), 
      label: d.mesh_label.value,
      leaf: false,
      parent: (d.parent_node == undefined ? "root" :  d.parent_node.value.replace(idPrefix, ""))
    });
    tree.push({
      id: d.mesh_id.value.replace(idPrefix, ""), 
      label: d.mesh_label.value,
      leaf: true,
      parent: d.node.value.replace(idPrefix, "")
    });
  });
  return tree;
};
```
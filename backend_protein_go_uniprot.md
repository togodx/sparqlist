# UniProt GO classification 

- backend SPARQLet
- Creators
  - 守屋

## Parameters

* `root` (type: GO) (Req.)
  * default: GO_0005575
  * example: GO_0008150 (biological process), GO_0005575 (cellular component), GO_0003674 (molecular function)

## Endpoint

https://rdfportal.org/sib/sparql

## `leaf`
- GO と UniProt のアノテーション関係
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://sparql.uniprot.org/uniprot>
FROM <http://sparql.uniprot.org/go>
WHERE {
    VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
    ?parent rdfs:subClassOf obo:{{root}} .  # <http://sparql.uniprot.org/go> ではすべて展開されているのでアスタリスク無し
    ?child up:classifiedWith ?parent ;
           up:proteome ?proteome .
    ?child up:mnemonic ?child_label .
    ?parent rdfs:label ?parent_label .
}
```

## `allLeaf`
- 全 UniProt (without annotation 用)
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?leaf ?leaf_label
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
  ?leaf a up:Protein ;
        up:mnemonic ?leaf_label ;
        up:proteome ?proteome .
}
```

## Endpoint

https://rdfportal.org/bioportal/sparql

## `graph`
- GO の親子関係
  - <http://sparql.uniprot.org/go> ではすべて展開されているので他のグラフを使用
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
SELECT DISTINCT ?parent ?child ?parent_label ?child_label
FROM <http://rdfportal.org/dataset/bioportal/go>
WHERE {
  ?child rdfs:subClassOf* obo:{{root}} ;
         rdfs:subClassOf ?parent .
  FILTER(REGEX(STR(?child), "GO_"))
  FILTER(REGEX(STR(?parent), "GO_"))
  ?child rdfs:label ?child_label .
  ?parent rdfs:label ?parent_label .
}
```

## `return`
```javascript
({root, leaf, graph, allLeaf}) => {
  const idPrefix = "http://purl.uniprot.org/uniprot/";
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
    if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label) tree[0].label = d.parent_label.value; // root の label 挿入
  })
  // アノテーション関係
  leaf.results.bindings.forEach(d => {
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
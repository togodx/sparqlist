# NCBI Gene GO annotation (chunked)

- backend SPARQLet
- GO term を分割して受け取り、ヒト NCBI Gene とのアノテーション関係を返す。
- BP (GO_0008150) は配下の term が 23,974 件あり、`ncbigene-go` 関係グラフ (約 2.5 億トリプル) との
  join がプラン出来ずクエリが返らないため、`subClassOf*` の展開を SPARQL から追い出して
  term リストを `VALUES` で渡す形にしている。
- 遺伝子シンボルはここでは返さない。呼び出し元が NCBI Gene RDF から取得して解決する。
- `ncbigene-go` は全生物種を含むため `ncbigene-taxonomy` でヒトに限定する。
  この絞り込みを外すとエンドポイントの上限 (1,000,000 行) で無警告に打ち切られる。
- 呼び出し元: backend_gene_go_ncbigene
- Creators
  - 池田
  - 守屋

## Parameters

* `terms` (type: list of GO ID) (Req.)
  * example: GO_0040011,GO_0006810,GO_0007610

## Endpoint

https://rdfportal.org/primary/sparql

## `input_terms`
```javascript
({ terms }) => {
  terms = terms.replace(/,/g, " ");
  if (terms.match(/\S/)) {
    return terms.split(/\s+/);
  }
  return [];
};
```

## `leaf`
- GO と NCBI Gene のアノテーション関係
- `?parent` は展開済みの term を受け取るので `subClassOf*` と GO オントロジのグラフは不要
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX tio: <http://togoid.dbcls.jp/ontology#>
PREFIX taxon: <http://identifiers.org/taxonomy/>
SELECT DISTINCT ?parent ?child
FROM <http://rdfportal.org/dataset/togoid/relation/ncbigene-go>
FROM <http://rdfportal.org/dataset/togoid/relation/ncbigene-taxonomy>
WHERE {
  VALUES ?parent { {{#each input_terms}} obo:{{this}} {{/each}} }
  ?child tio:TIO_000004 ?parent ;
         tio:TIO_000064 taxon:9606 .
}
```

## `return`
```javascript
({leaf}) => leaf.results.bindings;
```

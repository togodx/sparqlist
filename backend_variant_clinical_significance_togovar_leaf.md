# variant clinical significance (chunked)

- backend SPARQLet
- ClinVar variation ID の範囲 [`from`, `to`) を受け取り、TogoVar ID / rs ID / ClinVar clinical significance を返す。
- 全件を一括で取ると、ClinVar の variation ID と TogoVar の `dct:identifier` (数字のみの文字列) を
  文字列変換で join する部分が重く、proxy のタイムアウト (約 125 秒) を超えて返らない。
  `cvo:variation_id` (integer) の範囲で絞ると index が効くので、呼び出し元で範囲を分割して問い合わせる。
- 処理時間は返す行数にほぼ比例する (約 5,000 行/秒)。
- サブクエリで ClinVar 側を先に評価させる。外に出すと variant グラフ側から評価されて返らない。
- 呼び出し元: variant_clinical_significance_togovar
- Creators
  - 三橋
  - 守屋

## Parameters

* `from` (type: integer) (Req.)
  * example: 1000000
* `to` (type: integer) (Req.)
  * example: 1100000

## Endpoint

https://grch38.togovar.org/proxy/sparql

## `range`
```javascript
({ from, to }) => {
  return { from: parseInt(from, 10) || 0, to: parseInt(to, 10) || 0 };
};
```

## `leaf`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX cvo: <http://purl.jp/bio/10/clinvar/>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX sio: <http://semanticscience.org/resource/>

SELECT DISTINCT ?tgv_id ?rs_id ?category
WHERE {
  {
    SELECT DISTINCT ?var_id ?rs ?category
    WHERE {
      GRAPH <http://togovar.org/clinvar> {
        ?clinvar cvo:variation_id ?vid ;
          cvo:classified_record ?record .
        FILTER (?vid >= {{range.from}} && ?vid < {{range.to}})
        ?record sio:SIO_000628 / dct:references / rdfs:seeAlso ?rs ;
          cvo:rcv_list / cvo:rcv_accession / cvo:rcv_classifications /
            cvo:germline_classification / cvo:description / cvo:description ?category .
        FILTER (STRSTARTS(STR(?rs), "http://ncbi.nlm.nih.gov/snp/rs"))
      }
      BIND (STR(?vid) AS ?var_id)
    }
  }
  GRAPH <http://togovar.org/variant> {
    ?var dct:identifier ?var_id ;
      dct:identifier ?tgv_id .
    FILTER (STRSTARTS(?tgv_id, "tgv"))
  }
  BIND (STRAFTER(STR(?rs), "snp/") AS ?rs_id)
}
```

## `return`
```javascript
({leaf}) => leaf.results.bindings;
```

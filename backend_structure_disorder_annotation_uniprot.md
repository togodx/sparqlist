# UniProt disorder_annotation (chromosome 分割)

- backend SPARQLet
- 染色体ごとに disorder アノテーションを取得する。
- `VALUES ?proteome` に全 25 染色体を並べた状態で faldo の range 解決を join すると
  プランが破綻して返ってこないため、染色体単位に分割する。
  (chr21 単独なら 1.5 秒、全 25 染色体一括だと 300 秒でも無応答)
- 呼び出し元: structure_disorder_annotation_uniprot
- Creators
  - 井出
  - 守屋

## Parameters

* `chromosomes` (type: list of chromosome number) (Req.)
  * example: 21,22,X,Y,MT

## Endpoint

https://rdfportal.org/sib/sparql

## `input_chromosomes`
```javascript
({ chromosomes }) => {
  chromosomes = chromosomes.replace(/,/g, " ");
  if (!chromosomes.match(/\S/)) {
    return [];
  }
  // UP000005640 のプロテオームは MT だけ Mitochondrion で、他は Chromosome
  return chromosomes.split(/\s+/).map(c => {
    if (c === "MT") {
      return "Mitochondrion%20MT";
    } else if (c === "UP") {
      return "Unplaced";
    } else {
      return `Chromosome%20${c}`;
    }
  });
};
```

## `disorder`
- 呼び出し元の `results` はタンパク質単位で集計するので ORDER BY は不要
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
SELECT DISTINCT ?leaf ?label ?value ?seq_length ?begin_position
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
   VALUES ?proteome {
     {{#each input_chromosomes}} <http://purl.uniprot.org/proteomes/UP000005640#{{this}}> {{/each}}
   }
   ?leaf a up:Protein ;
         up:mnemonic ?label ;
         up:annotation ?annotation ;
         up:proteome ?proteome .
   ?annotation a up:Region_Annotation ;
               rdfs:comment "Disordered" ;
               up:range ?range .
   ?range rdf:type faldo:Region ;
          faldo:begin/faldo:position ?begin_position ;
          faldo:end/faldo:position ?end_position .
   ?range faldo:begin/faldo:reference/rdf:value ?seq_value .
   BIND (STRLEN(?seq_value) AS ?seq_length)
   BIND ((?end_position-?begin_position+1) AS ?value)
}
```

## `return`
```javascript
({disorder}) => disorder.results.bindings;
```

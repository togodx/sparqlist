# Ensembl transcript length 

SPARQL で取得するのは transcript - exon - length 関係のため、まとめて取ると 1,000,000 件制限に引っかかる。  
Chromosome ごとに分割して取るための backend SPARQList.  
v115 の 1 番染色体で、取得行数(transcript-exon の組み合わせのユニーク数)が 342949 となる。

## Parameters

* `chromosomes` (type: list of chromosome number) (Req.)
  * example: 21,22,X,Y,MT

## Description

- Data sources
    - Ensembl human: [https://useast.ensembl.org/Homo_sapiens/Info/Index](https://useast.ensembl.org/Homo_sapiens/Info/Index)
- Creators
  - 池田
  - 守屋

## Endpoint

https://rdfportal.org/ebi/sparql

## `input_chromosomes`
```javascript
({ chromosomes }) => {
  chromosomes = chromosomes.replace(/,/g, " ");
  if (chromosomes.match(/\S/)) {
    return chromosomes.split(/\s+/);
  }
};
```

## `data`
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX so: <http://purl.obolibrary.org/obo/so#>

SELECT DISTINCT ?child ?child_label ?exon_len
FROM <http://rdfportal.org/dataset/ensembl>
WHERE {
  ?enst so:transcribed_from ?ensg .
  ?ensg obo:RO_0002162 <http://ensembl.org/Homo_sapiens> ;
        so:part_of ?chr .
  ?enst dcterms:identifier ?child ;
        rdfs:label ?child_label ;
        so:has_part ?ense .
  ?ense faldo:location ?ense_location .
  VALUES ?chr {
    {{#each input_chromosomes}} <http://identifiers.org/hco/{{this}}/GRCh38> {{/each}}
  }
  #FILTER (REGEX (STR (?chr), "identifiers.org/hco/") && REGEX (STR (?chr), "GRCh38"))
  #BIND(STRBEFORE(STRAFTER(STR(?chr), "http://identifiers.org/hco/"), "/") as ?chromosome)
  #VALUES ?chromosome {
  #    "1" "2" "3" "4" "5" "6" "7" "8" "9" "10"
  #    "11" "12" "13" "14" "15" "16" "17" "18" "19" "20" "21" "22"
  #    "X" "Y" "MT"
  #}
  ?ense_location faldo:begin / faldo:position ?exon_start ;
                 faldo:end / faldo:position ?exon_end .
  BIND((xsd:integer(?exon_end) - xsd:integer(?exon_start) + 1) AS ?exon_len)
}
```
## `return`

```javascript
({data}) => {
  let len_sum = {};

  data.results.bindings.forEach(d => {
    if (!len_sum[d.child.value]) {
      len_sum[d.child.value] = {
        len: Number(d.exon_len.value),
        label: d.child_label.value
      };
    }
    else {
      len_sum[d.child.value].len += Number(d.exon_len.value);
    }
  });
  let transcripts = Object.keys(len_sum);
  const bin = 50;
  let tree = transcripts.map(t => {
    const num = bin * parseInt(len_sum[t].len/bin);
    const bin_label = num + "-" + (num + bin - 1);
    const bin_id = parseInt(len_sum[t].len/bin) + 1;
    return {
      id: t,
      label: len_sum[t].label,
      value: len_sum[t].len,
      binId: bin_id,
      binLabel: bin_label
    };
  });
  return tree;
};
```
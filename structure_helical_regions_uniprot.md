# UniProt helical regions

## Description
- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot/)
- Query
  - Input
    - UniProt ID
  - Output
    - Regions of Helix_Annotation in UniProt
- Creators
  - 井手・千葉

## Parameters
* `categoryIds` -(type:Annotation)
  * default: Helix_Annotation
  * example: Beta_Strand_Annotation

## `annotation_types`
- annotation 選択
```javascript
({ categoryIds }) => {
  categoryIds = categoryIds.replace(/,/g," ")
  if (categoryIds.match(/[^\s]/)) {
    return categoryIds.split(/\s+/);
  }
}
```

## Endpoint

https://rdfportal.org/sib/sparql

## `main`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?uniprot ?mnemonic ?seq_length ?region_length ?begin_position ?end_position
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?annotation_type { {{#each annotation_types}} up:{{this}} {{/each}} } 
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
  ?uniprot up:proteome ?proteome_subset ;
      up:mnemonic ?mnemonic;
      up:annotation ?annotation .
  ?annotation a ?annotation_type ;
      up:range ?range .
  ?range faldo:begin/faldo:position ?begin_position ;
         faldo:end/faldo:position ?end_position .
  ?range faldo:begin/faldo:reference/rdf:value ?seq .
  BIND(STRLEN(?seq) AS ?seq_length)
  BIND((?end_position - ?begin_position + 1) AS ?region_length)
}
ORDER BY ?uniprot ?begin_position
```

## `results`
```javascript
({ main }) => {
  let tree = [];
  let mnemonic = new Map();
  let seqLen = new Map();
  let regionLen = new Map();
  main.results.bindings.forEach(d => {
    const id = d.uniprot.value.replace('http://purl.uniprot.org/uniprot/', '');
    mnemonic.set(id, d.mnemonic.value);
    seqLen.set(id, Number(d.seq_length.value));
    if (regionLen.has(id)) {
      regionLen.set(id, Number(d.region_length.value) + regionLen.get(id));
    } else {
      regionLen.set(id, Number(d.region_length.value));
    }
  });
  regionLen.forEach((len, id) => {
    const pct = Math.round(len / seqLen.get(id) * 100);
    tree.push({
      id: id,
      label: mnemonic.get(id),
      value: pct,
      binId: pct + 1,
      binLabel: pct + "%"
    })
  });
  return tree;
}
```
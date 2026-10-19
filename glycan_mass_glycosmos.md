# Glycan mass 

## Description

- Data sources
  - GlyCosmos: [https://rdfportal.org/dataset/glycosmos/](https://rdfportal.org/dataset/glycosmos/)
- Query
  - Input
    - GlyTouCan ID
  - Output
    - Mass range
- Creators
  - 山本、池田

## Endpoint

https://rdfportal.org/primary/sparql

## `data`

```sparql
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX mass: <https://glycoinfo.gitlab.io/wurcsframework/org/glycoinfo/wurcsframework/1.0.1/wurcsframework-1.0.1.jar#>
PREFIX sbsmpt: <http://www.glycoinfo.org/glyco/owl/relation#>

SELECT DISTINCT ?mass ?glytoucan ?sbsmpt ?iupac
WHERE {
  ?wurcs mass:WURCSMassCalculator ?mass ;
         dcterms:source ?glytoucan ;
         a ?sbsmpt ;
         sbsmpt:subsumes* / dcterms:source / glycan:is_from_source / glycan:has_taxon <http://identifiers.org/taxonomy/9606> .
  OPTIONAL {
    ?glytoucan glycan:has_glycosequence [
      glycan:in_carbohydrate_format glycan:carbohydrate_format_iupac_condensed ;
      glycan:has_sequence ?iupac
    ] .
  }
}
```

## `return`
```javascript
({data}) => {
  const idPrefix = "http://rdf.glycoinfo.org/glycan/";

  let tree = [];
  data.results.bindings.forEach(d => {
    const bin = 200;
    const num = parseInt(Number(d.mass.value) / bin) * bin;
    const binLabel = num + "-" + (num + bin) + " Da";
    const binId = num / bin;
    const cap = 40;
    let label = "";
    if (d.iupac?.value) {
      label = d.iupac.value;
      if (label.length > cap) {
        label = label.substr(0, cap) + "...";
      }
    } else if (d.sbsmpt?.value) {
      label = "(" + d.sbsmpt.value.replace("http://www.glycoinfo.org/glyco/owl/relation#", "").replace(/_/g, " ") + ")";
    }
    tree.push({
      id: d.glytoucan.value.replace(idPrefix, ""),
      label: label,
      value: Number(d.mass.value),
      binId: binId,
      binLabel: binLabel
    });
  });

  return tree;
}
```
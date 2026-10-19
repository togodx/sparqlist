# Glycans in tissue

## Description

- Data sources
  - GlyCosmos: [https://rdfportal.org/dataset/glycosmos/](https://rdfportal.org/dataset/glycosmos/)
- Query
  - Input
    - GlyTouCan ID
  - Output
    - UBERON ID of tissues
- Creators
  - 山本、池田

## Endpoint

https://rdfportal.org/primary/sparql

## `data`

```sparql
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX info: <http://rdf.glycoinfo.org/glycan/>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX sbsmpt: <http://www.glycoinfo.org/glyco/owl/relation#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX sio: <http://semanticscience.org/resource/>

SELECT DISTINCT ?parent_label ?parent ?child ?child_label ?sbsmpt
WHERE {
  ?child sio:SIO_000255 [
     glycan:has_taxon <http://rdf.glycoinfo.org/source/9606> ;
     glycan:has_tissue ?parent
  ] .
  ?parent rdfs:label ?parent_label .
  ?wurcs dcterms:source ?child ;
         a ?sbsmpt ;
         sbsmpt:subsumes* / dcterms:source / glycan:is_from_source / glycan:has_taxon <http://identifiers.org/taxonomy/9606> .
  OPTIONAL {
    ?child glycan:has_glycosequence ?gsq .
    ?gsq glycan:in_carbohydrate_format glycan:carbohydrate_format_iupac_condensed ;
         glycan:has_sequence ?child_label .
  }
}
```

## `unclassified`
```sparql
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX info: <http://rdf.glycoinfo.org/glycan/>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX sbsmpt: <http://www.glycoinfo.org/glyco/owl/relation#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX sio: <http://semanticscience.org/resource/>

SELECT DISTINCT ?child ?child_label ?sbsmpt
WHERE {
  ?wurcs a ?sbsmpt ;
         dcterms:source ?child ;
         sbsmpt:subsumes* / dcterms:source / glycan:is_from_source / glycan:has_taxon <http://identifiers.org/taxonomy/9606> .
  VALUES ?sbsmpt { sbsmpt:Glycosidic_topology sbsmpt:Linkage_defined_saccharide }

  FILTER NOT EXISTS {
    ?child sio:SIO_000255 [
      glycan:has_taxon <http://rdf.glycoinfo.org/source/9606> ;
      glycan:has_tissue ?tissue
    ] .
  }
  OPTIONAL {
    ?child glycan:has_glycosequence ?gsq .
    ?gsq glycan:in_carbohydrate_format glycan:carbohydrate_format_iupac_condensed ;
         glycan:has_sequence ?child_label .
  }
}
```

## `return`

```javascript
({data, unclassified}) => {
  const parentIdPrefix = "http://purl.obolibrary.org/obo/";
  const childIdPrefix = "http://rdf.glycoinfo.org/glycan/";

  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  const cap = 40;
  function makeLabel(l, sbsmpt, cap) {
    let label = "";
    if (l) {
      label = l;
      if (label.length > cap) {
        label = label.substr(0, cap) + "...";
      }
    } else if (sbsmpt) {
      label = "(" + sbsmpt + ")";
    }
    return label;
  };
  // typed literal とそうでないのとで label が二重についているが、 SPARQL では除けないためここで uniq する
  let uniq = Array.from(
    data.results.bindings.reduce(
      (map, current) =>
      map.set(current.parent_label.value + "-" + current.child.value, current), new Map()).values());

  uniq.forEach(d => {
    if (!chk[d.parent.value]) {
      chk[d.parent.value] = true;
      tree.push({
        id: d.parent.value.replace(parentIdPrefix, ""),
        label: d.parent_label.value,
        leaf: false,
        parent: "root"
      })
    }

    tree.push({
      id: d.child.value.replace(childIdPrefix, ""),
      label: makeLabel(d.child_label?.value, d.sbsmpt?.value.replace("http://www.glycoinfo.org/glyco/owl/relation#", ""), cap),
      leaf: true,
      parent: d.parent.value.replace(parentIdPrefix, "")
    });
  });

  tree.push({
    id: "unclassified",
    label: "N/A",
    leaf: false,
    parent: "root"
  });

  unclassified.results.bindings.forEach(d => {
    if (!(d.child_label?.value) || d.child_label.type == "literal") {
    tree.push({
      id: d.child.value.replace(childIdPrefix, ""),
      label: makeLabel(d.child_label?.value, d.sbsmpt?.value.replace("http://www.glycoinfo.org/glyco/owl/relation#", ""), cap),
      leaf: true,
      parent: "unclassified"
    });
      }
  });

  return tree;
};
```
# Glycan subsumption

## Description

- Data sources
  - GlyCosmos: [https://rdfportal.org/dataset/glycosmos/](https://rdfportal.org/dataset/glycosmos/)
- Query
  - Input
    - GlyTouCan ID
  - Output
    - subsumption
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
PREFIX sbsmpt: <http://www.glycoinfo.org/glyco/owl/relation#>
PREFIX struct: <https://glytoucan.org/Structures/Glycans/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX info: <http://rdf.glycoinfo.org/glycan/>

SELECT DISTINCT ?parent ?child ?iupac
FROM <http://rdf.glytoucan.org/partner/glycome-db>
FROM <http://rdf.glycosmos.org/glycans/subsumption>
FROM <http://rdf.glycosmos.org/glycans/seq>
FROM <http://rdf.glycosmos.org/glycans/taxon>
WHERE {
  VALUES ?parent {
    sbsmpt:Linkage_defined_saccharide
    sbsmpt:Monosaccharide_composition_with_linkage
    sbsmpt:Glycosidic_topology
    sbsmpt:Base_composition_with_linkage
  }
  #VALUES ?child { info:G00155YT }
  ?wurcs a ?parent ;
         dcterms:source ?child .
  ?wurcs sbsmpt:subsumes* / dcterms:source / glycan:is_from_source / glycan:has_taxon <http://identifiers.org/taxonomy/9606> .
  OPTIONAL {
    ?child glycan:has_glycosequence [
      glycan:in_carbohydrate_format glycan:carbohydrate_format_iupac_condensed ;
      glycan:has_sequence ?iupac
    ] .
  }
}
```

## `return`

```javascript
({data}) => {
  const parentIdPrefix = "http://www.glycoinfo.org/glyco/owl/relation#";
  const childIdPrefix = "http://rdf.glycoinfo.org/glycan/";

  const idMap = new Map([["Base_composition_with_linkage", 1],
                         ["Monosaccharide_composition_with_linkage", 2],
                         ["Glycosidic_topology", 3],
                         ["Linkage_defined_saccharide", 4]]);
  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  const cap = 40;
  data.results.bindings.forEach(d => {
    let sbsmpt = d.parent.value.replace(parentIdPrefix, "")
    if (!chk[d.parent.value]) {
      chk[d.parent.value] = true;
      tree.push({
        id: idMap.get(sbsmpt).toString(),
        label: sbsmpt.replace(/_/g, " "),
        leaf: false,
        parent: "root"
      });
    }
    let label = "";
    if (d.iupac?.value) {
      label = d.iupac.value;
      if (label.length > cap) {
        label = label.substr(0, cap) + "...";
      }
    } else {
      label = "(" + sbsmpt.replace(/_/g, " ") + ")";
    }

    tree.push({
      id: d.child.value.replace(childIdPrefix, ""),
      label: label,
      leaf: true,
      parent: idMap.get(sbsmpt).toString()
    });
  });

  return tree;
}
```
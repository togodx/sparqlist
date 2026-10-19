# (TODO) Genes specifically expressed in cells (HPA)

- 理由：hpa_cell_specificity が RDF portal にない（？）

- Creators
  - 小野・池田・千葉

## Endpoint

https://rdfportal.org/primary/sparql

## `data`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>

SELECT DISTINCT ?parent ?parent_label ?child ?child_label
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/hpa_cell_specificity> {
    ?child refexo:isPositivelySpecificTo ?parent .
  }
  BIND(URI(REPLACE(STR(?child), "http://identifiers.org/ensembl/", "http://rdf.ebi.ac.uk/resource/ensembl/")) AS ?ebi_ensg)
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?ebi_ensg rdfs:label ?child_label .
  }
  ?parent rdfs:label ?parent_label .
}
```

## `lowSpec`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>

SELECT DISTINCT ?child ?child_label
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/hpa_cell_specificity> {
    ?child a refexo:HPA_ts_evaluated_gene .
    FILTER NOT EXISTS {
      ?child refexo:isPositivelySpecificTo ?t .
    }
  }
  BIND(URI(REPLACE(STR(?child), "http://identifiers.org/ensembl/", "http://rdf.ebi.ac.uk/resource/ensembl/")) AS ?ebi_ensg)
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
      ?ebi_ensg rdfs:label ?child_label .
  }
}
```

## `return`

```javascript
({data, lowSpec}) => {
  const childIdPrefix = "http://identifiers.org/ensembl/";
  const parentIdPrefix = "http://purl.obolibrary.org/obo/caloha.obo#"

  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  
  data.results.bindings.forEach(d => {
    if (!chk[d.parent.value]) {
      chk[d.parent.value] = true;
      tree.push({
        id: d.parent_label.value,
        label: d.parent_label.value,
        leaf: false,
        parent: "root"
      })
    }
    let label = d.child_label.value;
    if (label == "") {
      label = d.child.value.replace(childIdPrefix, "");
    }
    tree.push({
      id: d.child.value.replace(childIdPrefix, ""),
      label: label,
      leaf: true,
      parent: d.parent_label.value
    })
  });

  tree.push({     
    id: "unclassified",
    label: "Low specificity",
    leaf: false,
    parent: "root"
  });

  lowSpec.results.bindings.forEach(d => {
    let label = "(obsolete)";
    if (d.child_label) {
      label = d.child_label.value;
    }
    tree.push({
      id: d.child.value.replace(childIdPrefix, ""),
      label: label,
      leaf: true,
      parent: "unclassified"
    });
  });
  return tree;
};
```
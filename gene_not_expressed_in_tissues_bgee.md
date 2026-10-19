#  (TODO) Tissues where a gene is not expressed

## Description

- Data sources
  - Bgee
- Creators
  - 池田

## Endpoint

https://rdfportal.org/sib/sparql

## `data`
```sparql
PREFIX genex: <http://purl.org/genex#>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxonomy: <http://purl.uniprot.org/taxonomy/>

SELECT DISTINCT ?child_id ?child_label ?parent_id ?parent_label
FROM <http://bgee.org> 
{
	?seq a orth:Gene ;
         dcterms:identifier ?child_id ;
         genex:isAbsentIn ?anat ;
         rdfs:label ?child_label .
	?anat a genex:AnatomicalEntity ;
          rdfs:label ?parent_label .
	?seq orth:organism / obo:RO_0002162 taxonomy:9606 .
    BIND (STRAFTER(STR(?anat), "http://purl.obolibrary.org/obo/") AS ?parent_id)
    FILTER (STRSTARTS(?parent_id, "UBERON_"))
}
```

## `all`
```sparql
PREFIX genex: <http://purl.org/genex#>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxonomy: <http://purl.uniprot.org/taxonomy/>

SELECT DISTINCT ?child_id ?child_label {
  GRAPH <http://bgee.org> {    
    ?gene dcterms:identifier ?child_id ;
          rdfs:label ?child_label ;
          orth:organism / obo:RO_0002162 taxonomy:9606 .
  }
}
```

## `return`

```javascript
({data, all}) => {
  const idPrefix = "http://rdf.ebi.ac.uk/resource/ensembl/";

  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  let allEnsg = {};
  let unclassified = {};

  all.results.bindings.forEach(d => {
    let label = d.child_label.value;
    if (label == "") {
      label = d.child_id.value;
    }
    allEnsg[d.child_id.value] = label;
    unclassified[d.child_id.value] = true;
  });
  data.results.bindings.forEach(d => {
    if (!chk[d.parent_id.value]) {
      chk[d.parent_id.value] = true;
      tree.push({
        id: d.parent_id.value,
        label: d.parent_label.value,
        leaf: false,
        parent: "root"
      });
    }
    if (allEnsg[d.child_id.value]) {
      tree.push({
        id: d.child_id.value,
        label: d.child_label.value,
        leaf: true,
        parent: d.parent_id.value
      });
    }
    delete unclassified[d.child_id.value];
  });

  tree.push({     
    id: "unclassified",
    label: "Expressed in all tissues",
    leaf: false,
    parent: "root"
  });

  Object.keys(unclassified).forEach(d => {
    tree.push({
      id: d,
      label: allEnsg[d],
      leaf: true,
      parent: "unclassified"
    });
  });
  return tree;
};

```
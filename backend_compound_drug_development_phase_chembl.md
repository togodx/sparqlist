# ChEMBLを薬の開発フェーズで分類する

- Data sources
  - ChEMBL: [https://rdfportal.org/dataset/chembl/](https://rdfportal.org/dataset/chembl/)
- Creators
  - 信定、池田

## Endpoint

https://rdfportal.org/ebi/sparql

## Parameters
* `i` (The last digit of CHEMBL IDs: 0..9)

## `main`
```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?chembl ?parent ?child ?child_label
FROM <http://rdf.ebi.ac.uk/dataset/chembl>
WHERE
{
  ?chembl cco:chemblId ?child ;
          cco:substanceType ?type ;  # to select only ChEMBL compounds
          rdfs:label ?child_label .
  FILTER NOT EXISTS { ?chembl a cco:DrugIndication }
  OPTIONAL {
    ?chembl cco:highestDevelopmentPhase ?parent .  # When the phase is 0, this is not written in the RDF
  }
  FILTER(STRENDS(?child, '{{i}}'))
}
```
## `return`

```javascript
({ main }) => {
  let tree = [];

  main.results.bindings.forEach((elem) => {
    tree.push({
      id: elem.child.value,
      label: elem.child_label.value,
      leaf: true,
      parent: String(elem.parent?.value ?? 0),
   })
  });

  return tree;
};
```

# ChEBI Application での分類 

- Required SPARQLet: [backend_compound_chebi_classification_chebi](./backend_compound_chebi_classification_chebi)

## Description

- Data sources
  -  ChEIB: [https://rdfportal.org/dataset/chebi/](https://rdfportal.org/dataset/chebi/) 
- Query
  - Input
    - ChEBI id (number) for chemical compound(s)
  - Output
    -  ChEBI id (number) for application type(s) (subcategories of [Application (CHEBI:33232)](https://www.ebi.ac.uk/chebi/searchId.do?chebiId=CHEBI:33232) ) corresponding to the compound(s)
- Supplementary Information
  - The classification of compounds according to their application, defined in ChEBI ontology.
  - ChEBI Ontologyに定義された用途による、化合物の分類です。
- Creators
  - 川島、建石
    
## `httpreq`

```javascript
async ({})=>{
  let url = "backend_compound_chebi_classification_chebi"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body: 'root=33232',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
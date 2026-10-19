# ChEBI Chemical Role での分類 

- Required SPARQLet: [backend_compound_chebi_classification_chebi](./backend_compound_chebi_classification_chebi)

## Description

- Data sources
  -  ChEIB: [https://rdfportal.org/dataset/chebi/](https://rdfportal.org/dataset/chebi/) 
- Query
  - Input
    - ChEBI id (number)
  - Output
    -  [Chemical Role (CHEBI:51086)](https://www.ebi.ac.uk/chebi/searchId.do?chebiId=CHEBI:51086) and its subcategories
- Creators
  - 川島、建石、信定

## `httpreq`

```javascript
async ({})=>{
  let url = "backend_compound_chebi_classification_chebi"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body: 'root=51086',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
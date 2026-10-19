# ChEBI Biological Role での分類

- Required SPARQLet: [backend_compound_chebi_classification_chebi](./backend_compound_chebi_classification_chebi)

## Description

- Data sources
    -  [Chemical Entities of Biological Interest (ChEBI) ](https://www.ebi.ac.uk/chebi/) 
- Query
  - Input
    - ChEBI id (number)
  - Output
    -  [Biological Role (CHEBI:24432)](https://www.ebi.ac.uk/chebi/searchId.do?chebiId=CHEBI:24432) and its subcategories
- Creators
  - 川島、建石、信定

## `httpreq`

```javascript
async ({})=>{
  let url = "backend_compound_chebi_classification_chebi"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body: 'root=24432',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
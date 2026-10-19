# UniProt disulfide bond count distribution

- require : [backend_number_of_uniprot_annotation_uniprot](./backend_number_of_uniprot_annotation_uniprot)

## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot/)
- Query
  - Input
    - UniProt ID
  - Output
    - The number of disulfide bond
- Creators
  - 守屋

## `httpreq`

```javascript
async ({})=>{
  let url = "backend_number_of_uniprot_annotation_uniprot"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body: 'type=Disulfide_Bond_Annotation',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
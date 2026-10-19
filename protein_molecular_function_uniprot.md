# Gene Ontology 'Molecular function' distribution

- Required SPARQLet: [backend_protein_go_uniprot](./backend_protein_go_uniprot)

## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot/)
- Query
  - Input
    - UniProt ID
  - Output
    - Gene Ontology terms of biological process domain
- Creators
  - 守屋
  
## `httpreq`

```javascript
async ({})=>{
  let url = "backend_protein_go_uniprot"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body: 'root=GO_0003674',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
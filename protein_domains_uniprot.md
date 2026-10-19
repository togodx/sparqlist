# UniProt keywords 'Domain' distribution

- Required SPARQLet: [backend_protein_uniprot_keywords_uniprot](./backend_protein_uniprot_keywords_uniprot)
## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://www.uniprot.org/)
  - For details about UniProt Keywords, see [UniProt documentation](https://www.uniprot.org/help/keywords).

- Query
  - Input
    - UniProt ID
  - Output
    - UniProt Keyword in the Domain category
- Creators
  - 守屋

## `httpreq`

```javascript
async ({})=>{
  let url = "backend_protein_uniprot_keywords_uniprot"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body: 'root=9994',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
# Gene Ontology 'Cellular component' classification

- Required SPARQLet: backend_gene_go_ncbigene

## Description

- Data sources
  - NCBI Gene (TogoID)
    
- Query
  - Input
    - NCBI Gene ID
  - Output
    - Gene Ontology terms of cellular component domain
- Creators
  - 池田
  
## `httpreq`

```javascript
async ({})=>{
  let url = "backend_gene_go_ncbigene"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body: 'root=GO_0005575',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
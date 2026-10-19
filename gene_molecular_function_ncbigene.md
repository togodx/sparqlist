# Gene Ontology 'Molecular function' classification

- Required SPARQLet: backend_gene_go_ncbigene

## Description

- Data sources
  - NCBI Gene
- Query
  - Input
    - NCBI Gene ID
  - Output
    - Gene Ontology terms of molecular function domain
- Creators
  - 池田
  
## `httpreq`

```javascript
async ({})=>{
  let url = "backend_gene_go_ncbigene"; // parent SPARQLet relative path
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
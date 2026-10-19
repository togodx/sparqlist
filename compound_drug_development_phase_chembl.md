# ChEMBLを薬の開発フェーズで分類する

- require: [backend_compound_drug_development_phase_chembl](./backend_compound_drug_development_phase_chembl)
- 
## Description

- Data sources
  - ChEMBL: [https://rdfportal.org/dataset/chembl/](https://rdfportal.org/dataset/chembl/)
- Output
  - Highest development phase
- Creators
  - 信定、池田

## `return`

```javascript
async ({}) => {
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  // cf. https://chembl.gitbook.io/chembl-interface-documentation/frequently-asked-questions/drug-and-compound-questions
  tree.push({id: "-1.0", parent: "root", label: "-1: Unknown"});
  tree.push({id: "0", parent: "root", label: "0: Preclinical"});
  tree.push({id: "0.5", parent: "root", label: "0.5: Early Phase 1"});
  tree.push({id: "1.0", parent: "root", label: "1: PK tolerability"});
  tree.push({id: "2.0", parent: "root", label: "2: Efficacy"});
  tree.push({id: "3.0", parent: "root", label: "3: Safety & Efficacy"});
  tree.push({id: "4.0", parent: "root", label: "4: Indication Discovery & expansion"});

  let errors = [];
  const url = 'backend_compound_drug_development_phase_chembl';

  for (let i = 0; i <= 9; i++) {
  //for (let i = 11111; i <= 11112; i++) {
    let options = {
      method: 'POST',
      body: `i=${i}`,
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    };
    const results = await fetch(url, options).then((res) => {
      if (res.ok) {
        return res.json();
      } else {
        errors.push(i);
      }
    });

    if (results) {
      tree = tree.concat(results);
    }
  }

  if (errors.length) {
    tree.push({ errors: errors });
  }

  return tree;
}
```
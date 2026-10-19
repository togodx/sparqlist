# Ensembl transcript length 

- require: [backend_transcript_length_ensembl](./backend_transcript_length_ensembl)
- 染色体ごとに backend に問い合わせ、トランスクリプト長を取得する。

## Description

- Data sources
  - Ensembl human: [https://useast.ensembl.org/Homo_sapiens/Info/Index](https://useast.ensembl.org/Homo_sapiens/Info/Index)
- Creators
  - 池田

## `return`

```javascript
async ({}) => {
  let errors = [];
  let tree = [];
  const url = 'backend_transcript_length_ensembl';
  const chromosomes = ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10",
                       "11", "12", "13", "14", "15", "16", "17", "18", "19", "20",
                       "21", "22", "X", "Y", "MT"] ;
  for (const i of chromosomes) {
    let options = {
      method: 'POST',
      body: `chromosomes=${i}`,
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
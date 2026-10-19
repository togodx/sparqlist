# UniProt catalytic_activity

## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot/)
- Query
  - Input
    - UniProt ID
  - Output
    - The Catalytic activity based on EC number
- Creators
  - 井手、守屋

## Endpoint

https://rdfportal.org/sib/sparql

## `withAnnotation`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?leaf ?leaf_label ?parent 
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
   ?leaf a up:Protein ;
            up:mnemonic ?leaf_label;
            up:proteome ?proteome;
            up:annotation [
               a up:Catalytic_Activity_Annotation;
               up:catalyticActivity/up:enzymeClass ?parent
            ] .
}
```

## `results`

```javascript
({withAnnotation})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const parentPrefix = "http://purl.uniprot.org/enzyme/";

  let tree = [
    {id: "root", label: "Catalytic", root: true},  // {id: "root", label: "root node", root: true},  230602 ide fix
    {id: "10000000", label: "Oxidoreductases", leaf: false, parent: "root"},
    {id: "20000000", label: "Transferases",    leaf: false, parent: "root"},
    {id: "30000000", label: "Hydrolases",      leaf: false, parent: "root"},
    {id: "40000000", label: "Lyases",          leaf: false, parent: "root"},
    {id: "50000000", label: "Isomerases",      leaf: false, parent: "root"},
    {id: "60000000", label: "Ligase",          leaf: false, parent: "root"},
    {id: "70000000", label: "Translocase",     leaf: false, parent: "root"}
  ];

  let ec_chk = {};

  const calc_id = (ec_class) => {
    let num = parseInt(ec_class[0]) * 10000000;
    if (ec_class[1]) num += parseInt(ec_class[1]) * 100000;
    if (ec_class[2]) num += parseInt(ec_class[2]) * 1000;
    if (ec_class[3]) {
      if (ec_class[3].match(/^\d+$/)) num += parseInt(ec_class[3])
      else return num.toString() + ec_class[3];
    }
    return num.toString();    
  };

  const addGraph = (ec_class) => {
    const id = calc_id(ec_class);
    let ec = ec_class.join(".");
    if (!ec_chk[ec]) {
      ec_chk[ec] = true;
      if (ec_class.length < 2) ec += ".-";
      if (ec_class.length < 3) ec += ".-";
      if (ec_class.length < 4) ec += ".-";
      ec_class.pop();
      tree.push({
        id: id, 
        label: ec, 
        leaf: false, 
        parent: calc_id(ec_class)
      })
      if (ec_class.length > 1) addGraph(ec_class);
    }
    return id;
  };

  withAnnotation.results.bindings.forEach(d => {
    const parent_id = addGraph(d.parent.value.replace(parentPrefix, "").split("."));
    console.log(d.parent.value.replace(parentPrefix, "").split("."));
    tree.push({
      id: d.leaf.value.replace(idPrefix, ""),
      label: d.leaf_label.value,
      leaf: true,
      parent: parent_id
    })
  });
  return tree;
}
```
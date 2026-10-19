# UniProt pathway_annotation

## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot/)
- Query
  - Input
    - UniProt ID
  - Output
    - The pathway annotation in Uniprot (unipathway)  
- Creators
  - 井出

## Endpoint

https://rdfportal.org/sib/sparql

## `getpathway1`
```sparql
PREFIX upa: <http://purl.uniprot.org/unipathway/>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX skos:<http://www.w3.org/2004/02/skos/core#>
SELECT DISTINCT ?parent ?parent_label 
FROM <http://sparql.uniprot.org/pathways>
WHERE {
  ?parent rdf:type up:Pathway;
          rdfs:label ?parent_label;  
          rdfs:subClassOf ?root;
          skos:narrower ?child .     
  Filter(REGEX(?root,up:Pathway))
}
ORDER by ?parent
#limit 10
```

## `getpathway2`
```sparql
PREFIX upa: <http://purl.uniprot.org/unipathway/>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX skos:<http://www.w3.org/2004/02/skos/core#>
SELECT DISTINCT ?parent ?parent_label ?child  ?child_label 
FROM <http://sparql.uniprot.org/pathways>
WHERE {
  ?parent rdf:type up:Pathway;
          rdfs:label ?parent_label;  
          skos:narrower ?child .    
  ?child  rdfs:label ?child_label_temp .
  BIND((SUBSTR((STRAFTER(STR(?child_label_temp),STR(?parent_label))),3,99))AS ?child_label)
}
ORDER by ?parent
#limit 10
```

## `main`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX upa: <http://purl.uniprot.org/unipathway/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?leaf ?label ?parent  
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
   VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
   ?leaf a up:Protein;
            up:mnemonic ?label;
            up:annotation ?annotation;
            up:proteome ?proteome.
   ?annotation a up:Pathway_Annotation;
               rdfs:seeAlso ?parent .
}
#limit 10
```

## `results`

```javascript
({getpathway1, getpathway2, main})=>{
  const idPrefix_unipathway= "http://purl.uniprot.org/unipathway/";
  const idPrefix_uniprot="http://purl.uniprot.org/uniprot/";
  let tree = [
    {
      id: "root",
      //label: "root node", 230602 ide 
      label: "Pathway",
      root: true
    }
  ];
  getpathway1.results.bindings.forEach(d => {
    tree.push({
        id: d.parent.value.replace(idPrefix_unipathway, ""),
        label: d.parent_label.value,
        parent: "root"
      })
  });
  getpathway2.results.bindings.forEach(d => {
    tree.push({
        id: d.child.value.replace(idPrefix_unipathway, ""),
        label: d.child_label.value,
        parent: d.parent.value.replace(idPrefix_unipathway, "")
      })
  });
  main.results.bindings.forEach(d => {
    tree.push({
        id: d.leaf.value.replace(idPrefix_uniprot, ""),
        label: d.label.value,
        leaf: true,
        parent: d.parent.value.replace(idPrefix_unipathway, "")
      })
  });
  return tree;
}
```
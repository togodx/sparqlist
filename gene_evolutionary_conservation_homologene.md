# Evolutionary divergence

## Description
- Data sources
  - HomoloGene RDF: [https://rdfportal.org/dataset/homologene/](https://rdfportal.org/dataset/homologene/)
- Input/Output 
  - Input
    - NCBI Gene ID
  - Output
    - Organisms
- Supplementary information
  - On the basis of HomoloGene database including 21 eukaryotic species and divergence times between human and each species, the most distant organisms where each human gene is conserved are shown.
  - 真核生物21種を含むHomoloGeneデータベースおよびヒトと各生物種との分岐年代に基づいて、ヒト遺伝子が保存されている生物のうち最も遠縁の生物を表示しています。
- Creators
  - 千葉・池田

## Endpoint

https://rdfportal.org/primary/sparql


## `main`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX homologene: <https://ncbi.nlm.nih.gov/homologene/>
PREFIX taxid: <http://identifiers.org/taxonomy/>

SELECT DISTINCT ?human_gene ?gene_label ?branch_id ?branch_label
FROM <http://rdfportal.org/dataset/homologene>
WHERE {
  {
    SELECT ?human_gene ?gene_label (max(?branch) as ?branch_id)
    WHERE {
      ?grp orth:inDataset homologene: ;
          orth:hasHomologousMember ?human_gene, ?gene .
      ?human_gene obo:RO_0002162 taxid:9606 ;
          rdfs:label ?gene_label .
      ?gene obo:RO_0002162 ?taxid .
      VALUES (?taxid ?branch) {
        ( taxid:9606   1 )
        ( taxid:9598   2 )
        ( taxid:9544   3 )
        ( taxid:10090  4 )
        ( taxid:10116  4 )
        ( taxid:9615   5 )
        ( taxid:9913   5 )
        ( taxid:9031   6 )
        ( taxid:8364   7 )
        ( taxid:7955   8 )
        ( taxid:7227   9 )
        ( taxid:7165   9 )
        ( taxid:6239   9 )
        ( taxid:28985  10 )
        ( taxid:318829 10 )
        ( taxid:33169  10 )
        ( taxid:4896   10 )
        ( taxid:4932   10 )
        ( taxid:5141   10 )
        ( taxid:3702   11 )
        ( taxid:4530   11 )
      }
    }
  }
  VALUES (?branch_id ?branch_label) {
    ( 1  "human" )
    ( 2  "chimpanzee" )
    ( 3  "primates" )
    ( 4  "mouse, rat" )
    ( 5  "mammal" )
    ( 6  "chicken" )
    ( 7  "frog" )
    ( 8  "zebrafish" )
    ( 9  "insect, worm" )
    ( 10 "fungi" )
    ( 11 "plant" )
  }
}
```

## `return`

```javascript
({ main }) => {
   
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];

  let checked = {};

  main.results.bindings.forEach((elem) => {
    if (!checked[elem.branch_id.value]) {
      checked[elem.branch_id.value] = true;
      tree.push({
        id: 'branch_' + ('00' + elem.branch_id.value).slice(-2),
        label: capitalizeCsv(elem.branch_label.value),
        parent: "root",
      });
    }
    tree.push({
      id: elem.human_gene.value.replace("http://identifiers.org/ncbigene/", ""),
      label: elem.gene_label.value,
      leaf: true,
      parent: 'branch_' + ('00' + elem.branch_id.value).slice(-2),
    });
  });

  return tree;

  function capitalizeCsv(str) {
    return str.split(', ').map((s) => capitalize(s)).join(', ')
  }
  function capitalize(str) {
    return str.charAt(0).toUpperCase() + str.substring(1);
  }
};
```
# (TODO) IDG Development Level Summary

## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://www.uniprot.org/)
- Input/Output
  - Input
    - UniProt ID
  - Output
    - IDG Development level (Tclin/Tchem/Tbio/Tdark)
- Creators
  - 池田

[Illuminating the Druggable Genome (IDG)](https://commonfund.nih.gov/idg) プロジェクトで作られたデータベース [Pharos](https://pharos.nih.gov) における、薬剤標的としてのタンパク質の到達度の分類。

分類基準は以下の通り。 (Pharos の help より。help ページというものはないので、個別のページ ([例](https://pharos.nih.gov/targets/ITK)) の IDG Development Level Summary から help を参照。)

- Tclin
  - These targets have activities in DrugCentral (ie. approved drugs) with known mechanism of action.
- Tchem
  - These targets have activities in ChEMBL or DrugCentral that satisfy the activity thresholds detailed below. In some cases, targets have been manually migrated to Tchem by human curation based on small molecule activities from other sources.
- Tbio
  - These targets do not have known drug or small molecule activities that satisfy the activity thresholds detailed below AND satisfy one or more of the following criteria:
    - target is above the cutoff criteria for Tdark
    - target is annotated with a Gene Ontology Molecular Function or Biological Process leaf term(s) with an Experimental Evidence code
- Tdark
  - These are targets about which virtually nothing is known. They do not have known drug or small molecule activities that satisfy the activity thresholds detailed below AND satisfy two or more of the following criteria:
    - A PubMed text-mining score from Jensen Lab < 5
    - <= 3 Gene RIFs
    - <= 50 Antibodies available according to http://antibodypedia.com
- Activity Thresholds: Activity values from DrugCentral and ChEMBL must be standardizable to -Log Molar units AND meet the the following target-family-specific cutoffs:
  - GPCRs: <= 100nM
  - Kinases: <= 30nM
  - Ion Channels: <= 10μM
  - Non-IDG Family Targets: <= 1μM

情報は UniProt RDF から取得した。TrEMBL のデータは対象外のようなので、`withoutAnnotation` では `reviewed` かつ Pharos のアノテーションがないものを収集した。
## Endpoint

https://rdfportal.org/sib/sparql

## `withAnnotation`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX database: <http://purl.uniprot.org/database/>

SELECT DISTINCT ?child ?parent_label ?child_label
FROM <http://sparql.uniprot.org/uniprot>
FROM <http://sparql.uniprot.org/tissues>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
  ?child a up:Protein ;
         up:mnemonic ?child_label ;
         up:organism taxon:9606 ;
         up:proteome ?proteome ;
         rdfs:seeAlso ?pharos .
  ?pharos a up:Resource ;
          up:database database:Pharos ;
          rdfs:comment ?parent_label .
}
```

## `withoutAnnotation`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX database: <http://purl.uniprot.org/database/>

SELECT DISTINCT ?child ?child_label
FROM <http://sparql.uniprot.org/uniprot>
FROM <http://sparql.uniprot.org/tissues>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
  ?child a up:Protein ;
         up:mnemonic ?child_label ;
         up:organism taxon:9606 ;
         up:proteome ?proteome ;
         up:reviewed 1 .
  FILTER NOT EXISTS { ?child rdfs:seeAlso / up:database database:Pharos . }
}
```

## `return`
```javascript
({withAnnotation, withoutAnnotation})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const str2id = {
    "Tclin": "1",
    "Tchem": "2",
    "Tbio": "3",
    "Tdark": "4"
  }
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    },{
      id: "1",
      label: "Tclin",
      parent: "root"
    },{
      id: "2",
      label: "Tchem",
      parent: "root"
    },{
      id: "3",
      label: "Tbio",
      parent: "root"
    },{
      id: "4",
      label: "Tdark",
      parent: "root"
    },{
      id: "unclassified",
      label: "not annotated by Pharos",
      parent: "root"
    }
  ];

  // アノテーション関係
  withAnnotation.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: str2id[d.parent_label.value]
    })
  });
  // アノテーション無し要素
  withoutAnnotation.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value.replace(idPrefix, ""),
      label: d.child_label.value,
      leaf: true,
      parent: "unclassified"
    });
  })
  
  return tree;
}
```

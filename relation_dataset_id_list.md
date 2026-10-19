# dataset ID list for TogoDX human
- dataset の ID リストを返す
  - ensembl_gene, ncbigene, ensembl_transcript, uniprot, pdb, glytoucan は human 全体のリスト
  - それ以外は バーに乗ってる ID のリスト

## Parameters

* `dataset`
  * default: uniprot
  * example: ensembl_gene, ncbigene, ensembl_transcript, uniprot, pdb, chebi, chembl_compound, pubchem_compound, hmdb, glytoucan, mondo, mesh, nando, hp_phenotype, togovar

## `params`
```javascript
({dataset})=>{
  let endpoint = "https://rdfportal.org/primary/sparql";
  let query;
  if (dataset == "ensembl_gene") {
    endpoint = "https://rdfportal.org/ebi/sparql";
    query = `
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX tax: <http://identifiers.org/taxonomy/>
SELECT DISTINCT ?id
FROM <http://rdfportal.org/dataset/ensembl>
WHERE {
  [] dct:identifier ?id ;
     obo:RO_0002162 <http://ensembl.org/Homo_sapiens> .
}`;
  } else if (dataset =="ncbigene") {
    endpoint="https://rdfportal.org/ncbi/sparql";
    query = `
PREFIX insdc: <http://ddbj.nig.ac.jp/ontologies/nucleotide/>
PREFIX ncbio: <https://dbcls.github.io/ncbigene-rdf/ontology.ttl#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX tax: <http://identifiers.org/taxonomy/>
SELECT DISTINCT  ?id
FROM <http://rdfportal.org/dataset/ncbigene>
WHERE {
  [] a insdc:Gene ;
     dct:identifier ?id ;
     ncbio:taxid tax:9606 .
}`;
  } else if (dataset == "ensembl_transcript") {
    endpoint = "https://rdfportal.org/ebi/sparql";
    query = `
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX so: <http://purl.obolibrary.org/obo/so#>
PREFIX tax: <http://identifiers.org/taxonomy/>
SELECT DISTINCT ?id
FROM <http://rdfportal.org/dataset/ensembl>
WHERE {
  [] dct:identifier ?id ;
     so:transcribed_from [
       obo:RO_0002162 <http://ensembl.org/Homo_sapiens> 
     ] .
}`;
  } else if (dataset == "uniprot") {
    endpoint="https://rdfportal.org/sib/sparql";
    query = `
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX ch: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?id
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { ch:1 ch:2 ch:3 ch:4 ch:5 ch:6 ch:7 ch:8 ch:9 ch:10 
                     ch:11 ch:12 ch:13 ch:14 ch:15 ch:16 ch:17 ch:18 ch:19
                     ch:20 ch:21 ch:22 ch:X ch:Y mt:MT un: }
  ?uniprot a up:Protein ;
           up:proteome ?proteome .
  BIND(REPLACE(STR(?uniprot), "http://purl.uniprot.org/uniprot/", "") AS ?id)
}`;
  } else if (dataset == "pdb") {
    endpoint = "https://rdfportal.org/pdb/sparql";
    query = `
PREFIX pdbo: <http://rdf.wwpdb.org/schema/pdbx-with-vrptx-v50.owl#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX tax: <http://identifiers.org/taxonomy/>
SELECT DISTINCT ?id
FROM <http://rdfportal.org/dataset/pdbj>
WHERE{
 [] a pdbo:datablock ;
    dct:identifier ?id ;
      ( (pdbo:has_entity_src_genCategory /
         pdbo:has_entity_src_gen ) | 
       (pdbo:has_entity_src_natCategory /
        pdbo:has_entity_src_nat ) ) /
      rdfs:seeAlso tax:9606 .
}`;
  } else if (dataset == "chebi") {
    endpoint="https://rdfportal.org/ebi/sparql";
    query = `
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?id                          
FROM <http://rdf.ebi.ac.uk/dataset/chebi>
WHERE {
  # TogoDX human root 33232 (Application type), 24432 (Biological role), 51086 (Chemical role)
  VALUES ?root { obo:CHEBI_33232 obo:CHEBI_24432 obo:CHEBI_51086 }
  ?chebi a owl:Class ;
         rdfs:subClassOf [
      a owl:Restriction ;
      owl:onProperty obo:RO_0000087 ;
      owl:someValuesFrom / rdfs:subClassOf* ?root
    ] .
  BIND( STRAFTER( STR(?chebi), "/CHEBI_") AS ?id)
}`;
  } else if (dataset == "chembl_compound") {
    endpoint="https://rdfportal.org/ebi/sparql";
    query = `
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#> 
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
SELECT DISTINCT ?id
FROM <http://rdf.ebi.ac.uk/dataset/chembl>
WHERE {
  {
    {
      # ATC
      ?chembl cco:atcClassification [] .
    } UNION { 
      # action type 
      ?chembl ^cco:hasMolecule/rdf:type cco:Mechanism .
    }
  }  UNION {
    {
      # drug dev phase
      ?chembl cco:highestDevelopmentPhase ?development_phase .
      FILTER NOT EXISTS { ?chembl a cco:DrugIndication }
      FILTER (?development_phase != 0 )
    }  UNION {
      # drub mesh
      ?chembl cco:hasDrugIndication/cco:hasMesh [] .
    } 
  }
  ?chembl a cco:SmallMolecule ;
          cco:chemblId ?id . 
}`;
  } else if (dataset == "pubchem_compound") {
    endpoint = "https://rdfportal.org/pubchem/sparql";
    query = `
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
SELECT DISTINCT ?id
WHERE {
  [] a sio:CHEMINF_000562 ;
       sio:SIO_000011 ?cid .
      # dct:subject/rdf:type skos:concept . 
  BIND( STRAFTER (STR(?cid), "compound/CID") AS ?id)
}`;
  } else if (dataset == "hmdb") {
    query = `SELECT DISTINCT ?id
FROM <http://rdfportal.org/dataset/togoid/relation/chembl_compound-hmdb> 
WHERE {
  [] [] ?hmdb .  
  BIND( STRAFTER( STR(?hmdb), "/hmdb/") AS ?id)
}`;
  } else if (dataset == "glytoucan") {
    query = `
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX sbsmpt: <http://www.glycoinfo.org/glyco/owl/relation#>
SELECT DISTINCT ?id
WHERE {
  ?s rdfs:seeAlso ?glytoucan ;
     sbsmpt:subsumes*/ dct:source/ glycan:is_from_source / 
     glycan:has_taxon <http://identifiers.org/taxonomy/9606>  .
  BIND( STRAFTER( STR(?glytoucan), "Glycans/") AS ?id)
}`;
  } else if (dataset == "mondo") {
    query = `
PREFIX mondo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?id
FROM <http://rdfportal.org/ontology/mondo>
WHERE {
  ?mondo rdfs:subClassOf+ mondo:MONDO_0000001 .
  BIND( STRAFTER( STR(?mondo), "/MONDO_") AS ?id)
}`;
  } else if (dataset == "mesh") {
    endpoint="https://rdfportal.org/ncbi/sparql";
    query = `
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>
SELECT DISTINCT ?id
FROM <http://id.nlm.nih.gov/mesh>
WHERE {
  ?mesh meshv:treeNumber ?node .
  FILTER (contains(str(?node), "C"))
  BIND( STRAFTER( STR(?mesh), "/mesh/") AS ?id)
}`;
  } else if (dataset == "nando") {
    query = `
PREFIX nando: <http://nanbyodata.jp/ontology/NANDO_>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?id
FROM <http://nanbyodata.jp/ontology/nando>
WHERE {
  ?nando rdfs:subClassOf+ nando:0000001 .
  BIND( STRAFTER( STR(?nando), "/NANDO_") AS ?id)
}`;
  } else if (dataset == "hp_phenotype") {
    query = `
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?id
FROM <http://rdfportal.org/ontology/hp>
WHERE {
  ?hp rdfs:subClassOf+ obo:HP_0000118 .
  BIND( STRAFTER( STR(?hp), "/obo/HP_") AS ?id)
}`;
  } else if (dataset == "togovar") {
    endpoint = "https://grch38.togovar.org/sparql";
    query=`
PREFIX efo: <http://www.ebi.ac.uk/efo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX terms: <http://med2rdf.org/gwascatalog/terms/>
PREFIX gvo: <http://genome-variation.org/resource#>
PREFIX dct: <http://purl.org/dc/terms/>
SELECT DISTINCT ?id
FROM <http://togovar.org/efo>
FROM <http://togovar.org/gwas-catalog>
FROM <http://togovar.org/variant/annotation/ensembl>
FROM <http://togovar.org/variant>
FROM <http://togovar.org/variant/annotation/clinvar>
#FROM <http://togovar.org/clinvar>
WHERE {
  GRAPH <http://togovar.org/variant>{
    ?togovar dct:identifier ?id.
  }
  {
    [] terms:mapped_trait_uri/rdfs:subClassOf* efo:EFO_0000001 ;
       rdfs:seeAlso _:dbsnp .
    GRAPH <http://togovar.org/variant/annotation/ensembl> {
      ?togovar rdfs:seeAlso _:dbsnp .
    }
  } UNION {
    GRAPH <http://togovar.org/variant/annotation/clinvar> {
      ?togovar  dct:identifier ?rs_id .
    }
  }
}`;
  }
  return {endpoint: endpoint, query: query}; 
}
```

## Endpoint
{{params.endpoint}}

## `ids`
```sparql
{{params.query}}
```

## `return`
```javascript
({ids})=>{
  return ids.results.bindings.map(d => d.id.value);
}
```
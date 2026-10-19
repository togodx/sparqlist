# TogoID SPARQL API: route (multi) を生成して、一つの SPARQL で join
- https://integbio.jp/togosite/sparqlist/ から移植
- 応答速度やヒープサイズの関係で数百個ずつ
  - togocvar は 50 個くらいずつ
- 対応データセット
  - chebi, chembl_compound, chembl_target, ensembl_gene, ensembl_transcript, glytoucan, hgnc, hmdb, hp, mesh, medgen, mondo, nando, ncbigene, omim_phenotype, orphanet, togovar, uniprot
  - データセットを追加する場合は route 生成に追加して、SPARQL 部分の PREFIX を定義する

## Parameters

* `source` database
  * default: uniprot
* `target` database
  * default: chebi
* `ids`
  * default: Q9D7Q1,Q690N0,Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814

## `routes`

```javascript
({source, target}) => {
  let config = {
    database: {
      glycan: ["glytoucan"],
      variant: ["togovar"],
      gene: ["hgnc", "ncbigene", "ensembl_gene"],
      transcript: ["ensembl_transcript"],
      protein: ["uniprot", "chembl_target"],
      structure: ["pdb"],
      compound: ["pubchem_compound", "chembl_compound", "chebi", "hmdb"],
      nando: ["nando"],
      hp: ["hp_phenotype"],
      disease: ["mondo", "medgen", "omim_phenotype", "orphanet_phenotype", "mesh"]
    },
    route: {
      glycan: {
        glycan: [[]],
        variant: [["ncbigene"]],
        gene: [["ncbigene"]],
        transcript: [["ncbigene"]],
        protein: [["uniprot"]],
        structure: [["uniprot", "pdb"]],
        compound: [["pubchem_compound"]],
        nando: [["ncbigene", "medgen", "mondo"]],
        hp: [["ncbigene", "hgnc", "orphanet_gene", "orphanet_phenotype"], ["ncbigene", "uniprot", "omim_phenotype"]],
        disease: [["ncbigene", "medgen", "mondo"]]
      },
      variant: {
        variant: [[]],
        gene: [[]],
        transcript: [[]],
        protein: [["hgnc", "uniprot"]],
        structure: [["hgnc", "uniprot", "pdb"]],
        compound: [["hgnc", "uniprot", "reactome_reaction", "chebi"], ["hgnc", "uniprot", "chembl_target", "chembl_compound"]],
        nando: [["clinvar", "medgen", "mondo"]],
        hp: [["clinvar", "medgen"]],
        disease: [["clinvar", "medgen"]]
      },
      gene: {
        gene: [[], ["hgnc"]],
        transcript: [[]],
        protein: [["uniprot"]],
        structure: [["uniprot", "pdb"]],
        compound: [["uniprot", "reactome_reaction", "chebi"], ["uniprot", "chembl_target", "chembl_compound"]],
        nando: [["ncbigene", "medgen", "mondo"]],
        hp: [["hgnc", "orphanet_gene", "orphanet_phenotype"], ["uniprot", "omim_phenotype"]],
        disease: [["ncbigene", "medgen"]]
      },
      transcript: {
        transcript: [[]],
        protein: [["uniprot"]],
        structure: [["uniprot", "pdb"]],
        compound: [["uniprot", "reactome_reaction", "chebi"], ["uniprot", "chembl_target", "chembl_compound"]],
        nando: [["ncbigene", "medgen", "mondo"]],
        hp: [["hgnc", "orphanet_gene", "orphanet_phenotype"], ["uniprot", "omim_phenotype"]],
        disease: [["ncbigene", "medgen"]]
      },
      protein: {
        protein: [[]],
        structure: [[]],
        compound: [["reactome_reaction", "chebi"], ["chembl_target", "chembl_compound"]],
        nando: [["ncbigene", "medgen", "mondo"]],
        hp: [["uniprot", "hgnc", "orphanet_gene", "orphanet_phenotype"], ["uniprot", "omim_phenotype"]],
        disease: [["ncbigene", "medgen"]]
      },
      structure: {
        structure: [[]],
        compound: [["uniprot", "reactome_reaction", "chebi"], ["uniprot", "chembl_target", "chembl_compound"]],
        nando: [["uniprot", "ncbigene", "medgen", "mondo"]],
        hp: [["uniprot", "hgnc", "orphanet_gene", "orphanet_phenotype"], ["uniprot", "omim_phenotype"]],
        disease: [["uniprot", "ncbigene", "medgen"]]
      },
      compound: {
        compound: [[]],
        nando: [["chembl_compound", "mesh", "mondo"]],
        hp: [["chembl_compound", "mesh", "mondo", "medgen"], ["chembl_compound", "mesh", "mondo", "omim_phenotype"], ["chembl_compound", "mesh", "mondo", "orphanet"]],
        disease: [["chembl_compound", "mesh", "mondo"]],
      },
      nando: {
        nando: [[]],
        hp: [["mondo", "medgen"], ["mondo", "omim_phenotype"], ["mondo", "orphanet_phenotype"]],
        disease: [["mondo"]]
      },
      hp: {
        hp: [[]],
        disease: [["medgen", "mondo"], ["omim_phenotype", "mondo"], ["orphanet_phenotype", "mondo"]]
      },
      disease: {
        disease: [["mondo"]]
      }
    }
  };
 
  let sourceSubject, targetSubject;
  for (let subject of Object.keys(config.database)) {
    if (config.database[subject].includes(source)) sourceSubject = subject;
    if (config.database[subject].includes(target)) targetSubject = subject;
  }
  
  // 例外処理
  // chembl_compound - mesh
  if ((sourceSubject == "compound" && target == "mesh") 
      || (targetSubject == "compound" && source == "mesh")) {
    config.route.compound.disease[0].pop();
  }
  // chembl_compound - hmdd
  let hmdb = 0;
  if (source == "hmdb") {
    hmdb = 1;
    source = "chembl_compound"
  } else if (target == "hmdb") {
    hmdb = 2;
    target = "chembl_compound";
  }
  
  let makeRoute = (source, target, route) => {
    route.unshift(source);
    route.push(target);
    if (hmdb == 1) route.unshift("hmdb");
    else if (hmdb == 2) route.push("hmdb");
    return route.filter((x, i, self) => self.indexOf(x) === i);
  }
  
  let routes = [];
  for (let subject_1 of Object.keys(config.route)) {
    for (let subject_2 of Object.keys(config.route[subject_1])) {
      if (subject_1 == sourceSubject && subject_2 == targetSubject) {
        for (let route of config.route[subject_1][subject_2]) {
          routes.push(makeRoute(source, target, route));
        }
      } else if (subject_1 == targetSubject && subject_2 == sourceSubject) {
        for (let route of config.route[subject_1][subject_2]) {
          routes.push(makeRoute(source, target, route.reverse()));
        }
      }
    }
  }
  return routes;
}
```

## `pairListArray`
```javascript
({routes})=>{
  let array = [];
  for (let i = 0; i < routes.length; i++) {
    let route = routes[i];
    let list = [];
    for (let j = 0; j < route.length - 1; j++) {
      let id1 = "id_" + [j];
      let id2 = "id_" + [j + 1];
      if (j == 0) id1 = "source_uri";
      if (j == route.length - 2) id2 = "target_uri";
      list.push({source: route[j], target: route[j + 1], id1: id1, id2: id2});
    }
    let union = "UNION";
    if (i == routes.length - 1) union = "";
    array.push({list: list, union: union});
  }
  return array;
}
```

## `idList`

```javascript
({ids}) => {
  list = ids.replace(/,/g, ' ').trim();
  if (list.length > 0) {
    return list.split(/\s+/);
  }
  return false;
}
```

## Endpoint

https://rdfportal.org/primary/sparql

- http://ep.dbcls.jp/togoid/sparql
  - GRAPH: http://rdfportal.org/dataset/togoid/relation/ -> http://togoid.dbcls.jp/graph/

## Query `result`

```sparql
PREFIX chebi: <http://purl.obolibrary.org/obo/CHEBI_>
PREFIX chembl_compound: <http://identifiers.org/chembl.compound/>
PREFIX chembl_target: <http://identifiers.org/chembl.target/>
PREFIX ensembl_gene: <http://identifiers.org/ensembl/>
PREFIX ensembl_transcript: <http://identifiers.org/ensembl/>
PREFIX hgnc: <http://identifiers.org/hgnc/>
PREFIX hp_phenotype: <http://purl.obolibrary.org/obo/HP_>
PREFIX medgen: <http://identifiers.org/medgen/>
PREFIX mesh: <http://id.nlm.nih.gov/mesh/>
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX nando: <http://nanbyodata.jp/ontology/NANDO_>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX omim_phenotype: <http://identifiers.org/mim/>
PREFIX orphanet: <http://identifiers.org/orphanet.ordo/>
PREFIX pdb: <http://rdf.wwpdb.org/pdb/>
PREFIX pubchem_compound: <http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID>
PREFIX reactome_reaction: <http://identifiers.org/reactome/>
PREFIX togovar: <http://identifiers.org/togovar/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX glytoucan: <http://identifiers.org/glytoucan/>
PREFIX clinvar: <http://identifiers.org/clinvar/>
PREFIX hmdb: <http://identifiers.org/hmdb/>

SELECT DISTINCT ?source_id ?target_id #?source_uri ?target_uri
WHERE {
  VALUES ?source_uri { {{#each idList}} {{../source}}:{{this}} {{/each}} }
  {{#each pairListArray}}
  {
    SELECT DISTINCT ?source_uri ?target_uri
    WHERE {
    {{#each list}}
      {
        GRAPH <http://rdfportal.org/dataset/togoid/relation/{{source}}-{{target}}> {
          ?{{id1}} [] ?{{id2}} .
        }
      } UNION {
        GRAPH <http://rdfportal.org/dataset/togoid/relation/{{target}}-{{source}}> {
          ?{{id2}} [] ?{{id1}} .
        }
      }
    {{/each}}
    }
  } {{union}} 
  {{/each}}
  BIND (replace(str(?source_uri), {{source}}:, '') AS ?source_id)
  BIND (replace(str(?target_uri), {{target}}:, '') AS ?target_id)
}
```

## Return

```javascript
({result}) => {
  return result.results.bindings.map(data => {
    return Object.keys(data).reduce((obj, key) => {
      obj[key] = data[key].value;
      return obj;
    }, {});
  });
}
```
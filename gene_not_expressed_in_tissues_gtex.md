# (TODO) Tissues where a gene is not expressed

* 理由：tissue specificityがRDF portalにない（？）

## Description

- Data sources
  - [GTEx version 8](https://gtexportal.org/home/datasets)
- Creators
  - 池田

## Endpoint

{{SPARQLIST_TOGODX_SPARQL}}

## `data`
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX refexo:  <http://purl.jp/bio/01/refexo#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX schema: <http://schema.org/>
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>

SELECT DISTINCT ?parent_label ?parent ?child
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary> {
    ?refex a refexo:RefExEntry ;
           sio:SIO_000216 [
             a refexo:logTPMMax ;
             sio:SIO_000300 0
             #sio:SIO_000300 ?logtpmmax
           ] ;
           refexo:isMeasurementOf ?child ;
           refexo:refexSample ?refexs .
    #FILTER(?logtpmmax >= 0 && ?logtpmmax < 1)
  }
  
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary> {
    ?refexs dcterms:description ?parent_label ;
            dcterms:identifier ?parent .
  }
}
```

## `all`
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX so: <http://purl.obolibrary.org/obo/so#>
PREFIX terms: <http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX refexo:  <http://purl.jp/bio/01/refexo#>

SELECT DISTINCT ?child ?child_label ?type
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary> {
    ?refex a refexo:RefExEntry ;
           refexo:isMeasurementOf ?child .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?child terms:has_biotype ?type ;
           rdfs:label ?child_label .
    [] so:transcribed_from ?child .
  }
}
```

## `return`

```javascript
({data, all}) => {
  const idPrefix = "http://rdf.ebi.ac.uk/resource/ensembl/";

  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  let allEnsg = {};
  let unclassified = {};
  // GTEx のプロトコルの原理上、small RNA は取れていないはずだが、GTEx のデータ上には存在してしまっている。
  // そのような RNA を GTEx のデータを用いて「無発現」と判断するべきでないので、除く。
  // "Protein coding", "Long non-coding RNA (lncRNA)", "Processed pseudogene",
  // "Unprocessed pseudogene", "Unitary pseudogene", "IG pseudogene", "TEC (To be Experimentally Confirmed)",
  // "IG V gene", "IG D gene", "IG J gene", "IG C gene", "TR V gene", "TR D gene", "TR J gene", "TR C gene",
  // "mt_gene", "IG_C_pseudogene", "IG_J_pseudogene", "IG_V_pseudogene", "TR_V_pseudogene", "TR_J_pseudogene",
  // "translated_processed_pseudogene", "transcribed_unprocessed_pseudogene",
  // "transcribed_unitary_pseudogene", "transcribed_processed_pseudogene"

  const biotypes_ensgloss = ["26", "28", "48", "49", "53", "54", "57", "59", "60", "61", "62", "63", "64", "65", "66"].map((x)=>{return "http://ensembl.org/glossary/ENSGLOSSARY_00000" + x});
  const biotypes_so = ["0088", "2100", "2101", "2102", "2103", "2104", "2105", "2107", "2108", "2109"].map((x)=>{return "http://purl.obolibrary.org/obo/SO_000" + x});
  const biotypes = new Set(biotypes_so.concat(biotypes_ensgloss))
 
  all.results.bindings.forEach(d => {
    let label = d.child_label.value;
    if (label == "") {
      label = d.child.value.replace(idPrefix, "");
    }
    if (biotypes.has(d.type.value)) {
      allEnsg[d.child.value] = label;
      unclassified[d.child.value] = true;
    }
  });
  data.results.bindings.forEach(d => {
    if (!chk[d.parent.value]) {
      chk[d.parent.value] = true;
      tree.push({     
        id: d.parent_label.value,
        label: d.parent_label.value,
        leaf: false,
        parent: "root"
      });
    }
    if (allEnsg[d.child.value]) {
      tree.push({
        id: d.child.value.replace(idPrefix, ""),
        label: allEnsg[d.child.value],
        leaf: true,
        parent: d.parent_label.value
      });
    }
    delete unclassified[d.child.value];
  });

  tree.push({     
    id: "unclassified",
    label: "Expressed in all tissues",
    leaf: false,
    parent: "root"
  });

  Object.keys(unclassified).forEach(d => {
    tree.push({
      id: d.replace(idPrefix, ""),
      label: allEnsg[d],
      leaf: true,
      parent: "unclassified"
    });
  });
  return tree;
};

```
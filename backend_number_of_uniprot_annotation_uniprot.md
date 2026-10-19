# UniProt annotation count distribution

- backend
- Creators
  - 守屋

- human にある annotation (
core:Absorption_Annotation
core:Active_Site_Annotation
core:Activity_Regulation_Annotation
core:Allergen_Annotation
core:Alternative_Initiation_Annotation
core:Alternative_Promoter_Usage_Annotation
core:Alternative_Sequence_Annotation
core:Alternative_Splicing_Annotation
core:Annotation
*core:Beta_Strand_Annotation
core:Binding_Site_Annotation
core:Biotechnology_Annotation
core:Calcium_Binding_Annotation
core:Catalytic_Activity_Annotation
core:Caution_Annotation
core:Chain_Annotation
core:Cofactor_Annotation
core:Coiled_Coil_Annotation
core:Compositional_Bias_Annotation
core:Cross-link_Annotation
core:Developmental_Stage_Annotation
core:Disease_Annotation
*core:Disulfide_Bond_Annotation
core:Domain_Annotation
core:Domain_Extent_Annotation
core:Erroneous_Gene_Model_Prediction_Annotation
core:Erroneous_Initiation_Annotation
core:Erroneous_Termination_Annotation
core:Erroneous_Translation_Annotation
core:Frameshift_Annotation
core:Function_Annotation
core:Glycosylation_Annotation
*core:Helix_Annotation
core:Induction_Annotation
core:Initiator_Methionine_Annotation
core:Intramembrane_Annotation
core:Kinetics_Annotation
core:Lipidation_Annotation
core:Mass_Spectrometry_Annotation
core:Metal_Binding_Annotation
core:Modified_Residue_Annotation
core:Motif_Annotation
core:Mutagenesis_Annotation
core:NP_Binding_Annotation
core:Natural_Variant_Annotation
core:Non-adjacent_Residues_Annotation
core:Non-standard_Residue_Annotation
core:Non-terminal_Residue_Annotation
core:Nucleotide_Binding_Annotation
core:PH_Dependence_Annotation
core:PTM_Annotation
core:Pathway_Annotation
core:Peptide_Annotation
core:Pharmaceutical_Annotation
core:Polymorphism_Annotation
core:Propeptide_Annotation
core:RNA_Editing_Annotation
core:Redox_Potential_Annotation
core:Region_Annotation
core:Repeat_Annotation
core:Ribosomal_Frameshifting
core:Sequence_Caution_Annotation
core:Sequence_Conflict_Annotation
core:Sequence_Uncertainty_Annotation
core:Signal_Peptide_Annotation
core:Similarity_Annotation
core:Site_Annotation
core:Subcellular_Location_Annotation
core:Subunit_Annotation
core:Temperature_Dependence_Annotation
core:Tissue_Specificity_Annotation
core:Topological_Domain_Annotation
core:Transit_Peptide_Annotation
core:Transmembrane_Annotation
*core:Turn_Annotation
core:Zinc_Finger_Annotation
 )
 
## Parameters

* `type`
  * default: Helix_Annotation
* `region_annotation`
  * example: Disordered

## `query_code`
```javascript
({type, region_annotation}) => {
  if (type == "Region_Annotation" && region_annotation) {
    return "rdfs:comment \"" + region_annotation + "\" ;";
  } else {
    return "";
  }
}
```

## Endpoint

https://rdfportal.org/sib/sparql

## `withAnnotation`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?leaf ?label (COUNT (DISTINCT ?annotation) AS ?value)
FROM <http://sparql.uniprot.org/uniprot>
WHERE
{
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
  ?leaf a core:Protein ;
         core:mnemonic ?label ;
         core:proteome ?proteome ;
         core:annotation ?annotation .
  ?annotation {{query_code}}
              a core:{{type}} .
}
```

## `withoutAnnotation`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?leaf ?label ?value
FROM <http://sparql.uniprot.org/uniprot>
WHERE {
  VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
  ?leaf a core:Protein ;
           core:mnemonic ?label ;
           core:proteome ?proteome .
  MINUS {
    ?leaf core:annotation [ 
      {{query_code}} 
      a core:{{type}}
    ] .
  }
  BIND ("0" AS ?value)
}
```

## `results`

```javascript
({withAnnotation, withoutAnnotation})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  
  return withAnnotation.results.bindings.concat(withoutAnnotation.results.bindings).map(d => {
    return {
      id: d.leaf.value.replace(idPrefix, ""),
      label: d.label.value,
      value: Number(d.value.value),
      binId: Number(d.value.value) + 1,
      binLabel: d.value.value
    }
  });
}
```
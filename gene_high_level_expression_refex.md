# RefEx organ specific 'high' expression distribution

- Required SPARQLet: backend_gene_expression_level_refex

## Description

- Data sources
    - [https://rdfportal.org/dataset/refex/](https://rdfportal.org/dataset/refex/)
    - [Calculations for tissue specificity of genechip human GSE7307](https://doi.org/10.6084/m9.figshare.4028700.v3) from [RefEx \(Reference Expression dataset\)](https://refex.dbcls.jp/)
- Input/Output
  - Input
    - NCBI Gene ID
  - Output
    - [RefEx tissue name classification table of 40 organs](https://doi.org/10.6084/m9.figshare.4028718.v5)
- Supplementary information
  - The tissue-specific highly expressed genes provided in this attribute are data available from [RefEx](https://refex.dbcls.jp) and are calculated for 34 organ samples in the original gene expression data ([GSE7307](https://www.ncbi.nlm.nih.gov /geo/query/acc.cgi?acc=GSE7307)). Tissue specificity of each gene was calculated using the ROKU method [(Kadota et al., BMC Bioinformatics, 2006, 7:294)](https://doi.org/10.1186/1471-2105-7-294), with 1 for high expression outliers, -1 for low expression outliers, and 0 for non-outliers. Tissue-specific highly expressed genes provided here refer to genes flagged as "1" as outliers that are highly expressed only in a single organ.
  - ここで提供している組織特異的な高発現遺伝子は、オリジナルの遺伝子発現データ([GSE7307](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE7307))に含まれる34臓器のサンプルに対して、ROKU法[(Kadota et al., BMC Bioinformatics, 2006, 7:294)](https://doi.org/10.1186/1471-2105-7-294)を用いて、高発現の外れ値に1、低発現の外れ値に-1、非外れ値に0からなる組織特異性行列を計算した結果です。ROKU法の技術的詳細は[こちらをご覧ください(22-24ページ) ](http://bioconductor.org/packages/release/bioc/manuals/TCC/man/TCC.pdf) 。ここで提供している組織特異的な高発現遺伝子とは、単一の器官でのみ高発現している外れ値として「1」のフラグが立てられた遺伝子を示します。
- Creators
  - 守屋

## `httpreq`

```javascript
async ({})=>{
  const url = "backend_gene_expression_level_refex"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
  
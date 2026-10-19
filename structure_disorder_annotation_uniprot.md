# UniProt disorder_annotation

## Description

- Data sources
  - UniProt: [https://rdfportal.org/dataset/uniprot/](https://rdfportal.org/dataset/uniprot/)
- Query
  - Input
    - UniProt ID
  - Output
    - The disorder annotation in Region_Annotation of Uniprot
- Required SPARQLet: backend_structure_disorder_annotation_uniprot
- `disorder` は染色体ごとに backend に問い合わせる。全 25 染色体を一括で投げると
  faldo の range 解決の join がプラン出来ず返ってこないため。
- Creators
  - 井出
  - 守屋

## Endpoint

https://rdfportal.org/sib/sparql

## `disorder`
- 染色体ごとに backend_structure_disorder_annotation_uniprot へ問い合わせて連結する
```javascript
async ({}) => {
  const url = "backend_structure_disorder_annotation_uniprot"; // child SPARQLet relative path
  const chromosomes = ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10",
                       "11", "12", "13", "14", "15", "16", "17", "18", "19", "20",
                       "21", "22", "X", "Y", "MT", "UP"];
  const CONCURRENCY = 4;
  const TIMEOUT_MS = 120000;

  // SPARQList の JS サンドボックスには AbortSignal が無いので Promise.race で実装し、
  // setTimeout も無い環境ではタイムアウト無しに degrade させる
  const hasTimer = typeof setTimeout === 'function';
  const fetchWithTimeout = (options) => {
    if (!hasTimer) return fetch(url, options);
    let timer;
    const req = fetch(url, options).finally(() => clearTimeout(timer));
    req.catch(() => {}); // race に負けた側の rejection を未処理にしない
    return Promise.race([
      req,
      new Promise((_, reject) => {
        timer = setTimeout(() => reject(new Error(`timeout after ${TIMEOUT_MS}ms`)), TIMEOUT_MS);
      })
    ]);
  };

  const fetchChromosome = async (chromosome, retry = 2) => {
    let options = {
      method: 'POST',
      body: `chromosomes=${chromosome}`,
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    };
    try {
      const res = await fetchWithTimeout(options);
      if (!res.ok) throw new Error(`status ${res.status}`);
      return await res.json();
    } catch (e) {
      // 一部の染色体が欠けた結果を黙って返さないよう、リトライしても駄目なら中断する
      if (retry > 0) return fetchChromosome(chromosome, retry - 1);
      throw new Error(`${url} failed (chromosome ${chromosome}): ${e.message}`);
    }
  };

  let bindings = [];
  for (let i = 0; i < chromosomes.length; i += CONCURRENCY) {
    const batch = await Promise.all(chromosomes.slice(i, i + CONCURRENCY).map(c => fetchChromosome(c)));
    batch.forEach(b => { bindings = bindings.concat(b); });
  }

  // 後続の results が SPARQL 結果と同じ形で扱えるようにする
  return {results: {bindings: bindings}};
};
```

## `withoutdisorder`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX chr: <http://purl.uniprot.org/proteomes/UP000005640#Chromosome%20>
PREFIX mt: <http://purl.uniprot.org/proteomes/UP000005640#Mitochondrion%20>
PREFIX un: <http://purl.uniprot.org/proteomes/UP000005640#Unplaced>
SELECT DISTINCT ?leaf ?label ?value
FROM <http://sparql.uniprot.org/uniprot>
 WHERE {
   VALUES ?proteome { chr:1 chr:2 chr:3 chr:4 chr:5 chr:6 chr:7 chr:8 chr:9 chr:10 
                   chr:11 chr:12 chr:13 chr:14 chr:15 chr:16 chr:17 chr:18 chr:19 chr:20 
                   chr:21 chr:22 chr:X chr:Y mt:MT un: }
   ?leaf a up:Protein ;
         up:mnemonic ?label ;
   		 up:proteome ?proteome .
   MINUS {
    ?leaf up:annotation ?annotation .
    ?annotation  a up:Region_Annotation ;
                 rdfs:comment "Disordered".
   }
  BIND ("0" AS ?value)
}
#limit 10
```

## `results`

```javascript
({disorder,withoutdisorder})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  let tree = [];

  // タンパク質ごとに disorder 領域長を合計する。
  // 染色体ごとに分割取得するため同一タンパク質の行が連続する保証が無く、また
  // 複数染色体に座乗するタンパク質 (ヒストン等 7 件) は同じ領域が重複して届くので、
  // 行の並びに依存せず領域単位で重複を除いて積算する
  let summary = {};
  disorder.results.bindings.forEach(d => {
    const id = d.leaf.value.replace(idPrefix, "");
    if (!summary[id]) {
      summary[id] = {
        label: d.label.value,
        seq_length: Number(d.seq_length.value),
        total: 0,
        regions: {}
      };
    }
    const region = d.begin_position.value + "-" + d.value.value;
    if (summary[id].regions[region]) return;
    summary[id].regions[region] = true;
    summary[id].total += Number(d.value.value);
  });
  Object.keys(summary).forEach(id => {
    const num = parseInt(100 * summary[id].total / summary[id].seq_length);
    tree.push({
      id: id,
      label: summary[id].label,
      value: num,
      binId: num + 1,
      binLabel: num + "%"
    })
  });

  withoutdisorder.results.bindings.forEach(f => {
    tree.push({
      id: f.leaf.value.replace(idPrefix, ""),
      label: f.label.value,
      value: Number(f.value.value),
      binId: 1,
      binLabel: f.value.value + "%"
    })
  });
  return tree;
}
```

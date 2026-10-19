# variant clinical significance

## Description

- Data sources
  - [TogoVar](https://togovar.org/?) (limited to variants with frequency data in Japanese populations)
- Query
  - Input
    - TogoVar id
  - Output
    - [Clinical significance of ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/docs/clinsig/)
- Required SPARQLet: backend_variant_clinical_significance_togovar_leaf
- Creators
  - 三橋
  - 守屋

## Endpoint

https://grch38.togovar.org/proxy/sparql

## `maxId`
- ClinVar variation ID の最大値 (分割範囲の上限)
```sparql
PREFIX cvo: <http://purl.jp/bio/10/clinvar/>
SELECT (MAX(?vid) AS ?max)
WHERE {
  GRAPH <http://togovar.org/clinvar> {
    ?clinvar cvo:variation_id ?vid .
  }
}
```

## `leaf`
- ClinVar と TogoVar の対応付け
- 一括で取ると proxy のタイムアウト (約 125 秒) を超えて返らないため、
  ClinVar variation ID の範囲で分割して backend_variant_clinical_significance_togovar_leaf に問い合わせる
- 1 チャンクの処理時間は行数にほぼ比例 (約 5,000 行/秒)。ID が小さい範囲ほど行数が多く、
  200,000 ID で約 20 秒
```javascript
async ({maxId}) => {
  const url = "backend_variant_clinical_significance_togovar_leaf"; // child SPARQLet relative path
  const CHUNK_SIZE = 200000;
  const CONCURRENCY = 4;
  const TIMEOUT_MS = 120000;

  const max = parseInt(maxId.results.bindings[0].max.value, 10);
  let ranges = [];
  for (let from = 0; from <= max; from += CHUNK_SIZE) {
    ranges.push([from, from + CHUNK_SIZE]);
  }

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

  const fetchRange = async ([from, to], retry = 2) => {
    let options = {
      method: 'POST',
      body: `from=${from}&to=${to}`,
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
      // 部分的に欠けた木を黙って返さないよう、リトライしても駄目なら中断する
      if (retry > 0) return fetchRange([from, to], retry - 1);
      throw new Error(`${url} failed (variation ID ${from}-${to}): ${e.message}`);
    }
  };

  let bindings = [];
  for (let i = 0; i < ranges.length; i += CONCURRENCY) {
    const batch = await Promise.all(ranges.slice(i, i + CONCURRENCY).map(r => fetchRange(r)));
    batch.forEach(b => { bindings = bindings.concat(b); });
  }

  // 後続の return が SPARQL 結果と同じ形で扱えるようにする
  return {results: {bindings: bindings}};
};
```

## `return`

```javascript
({leaf}) => {  
  const childLabelPrefix = "http://identifiers.org/dbsnp/";
  let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];  
  
  // アノテーション関係
  let clinsig = {};
  leaf.results.bindings.forEach(d => {
    d.category.value.split(/[;\/]/).forEach(sig_label => {
      const parent = sig_label.toLowerCase().replace(/,?\s+/g, "_");
      if (!clinsig[parent]) { // significanceをrootの下に
        clinsig[parent] = true;
        tree.push({
           id: parent,
           label: sig_label,
           parent: "root"
        });
      }
      tree.push({
        id: d.tgv_id.value,
        label: d.rs_id.value,
        leaf: true,
        parent: parent
      });
    });
  });
  return tree;	
}
```
# NCBI Gene GO classification

- backend SPARQLet
- Required SPARQLet: backend_gene_go_ncbigene_leaf
- `leaf` (GO - NCBI Gene のアノテーション関係) は GO term を 2,000 件ずつに分割して
  backend_gene_go_ncbigene_leaf に問い合わせる。BP (GO_0008150) は配下の term が
  23,974 件あり、`subClassOf*` の展開と `ncbigene-go` 関係グラフ (約 2.5 億トリプル) の
  join がプラン出来ず、一括ではクエリが返ってこないため。
- 遺伝子シンボルは NCBI Gene RDF の `rdfs:label` から取得する。TogoID の hgnc_symbol 経由だと
  HGNC 承認シンボルを持つ 44,946 件のみになり、`LOC*` 系 (未命名遺伝子) が約 2 万件落ちるため。
- Creators
  - 池田
  - 守屋

## Parameters

* `root` (type: GO) (Req.)
  * default: GO_0005575
  * example: GO_0008150 (biological process), GO_0005575 (cellular component), GO_0003674 (molecular function)

## Endpoint

https://rdfportal.org/primary/sparql

## `graph`
- GO の親子関係
- ここで得られる term 集合が root 配下の全 term (BP なら 23,974 件) と一致するため、
  `terms` セクションで抽出して `leaf` の分割に使う
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?parent ?child ?child_label ?parent_label
FROM <http://rdfportal.org/ontology/go>
WHERE {
  ?child rdfs:subClassOf* obo:{{root}} ;
  #?child rdfs:subClassOf* obo:GO_0008150 ;
         rdfs:subClassOf ?parent .
  FILTER(REGEX(STR(?child), "GO_"))
  FILTER(REGEX(STR(?parent), "GO_"))
  ?child rdfs:label ?child_label .
  ?parent rdfs:label ?parent_label .
}
```

## `terms`
- `graph` の結果から root 配下の GO term ID を抽出
```javascript
({root, graph}) => {
  const categoryPrefix = "http://purl.obolibrary.org/obo/";
  const terms = new Set([root]); // root 直下のアノテーション (evidence code ND) も拾うため root を含める
  graph.results.bindings.forEach(d => {
    terms.add(d.child.value.replace(categoryPrefix, ""));
    terms.add(d.parent.value.replace(categoryPrefix, ""));
  });
  return [...terms];
};
```

## `leaf`
- GO と NCBI Gene のアノテーション関係
- 一括で取ると BP (23,974 term) では返ってこないため、backend_gene_go_ncbigene_leaf に分割して問い合わせる
- CHUNK_SIZE は POST body のサイズ上限 (100KB) に依る。2,000 term で約 35KB
```javascript
async ({terms}) => {
  const url = "backend_gene_go_ncbigene_leaf"; // child SPARQLet relative path
  const CHUNK_SIZE = 2000;
  const CONCURRENCY = 4;
  const TIMEOUT_MS = 120000;

  let chunks = [];
  for (let i = 0; i < terms.length; i += CHUNK_SIZE) {
    chunks.push(terms.slice(i, i + CHUNK_SIZE));
  }

  // エンドポイントが稀に極端に遅い実行プランを選ぶことがある。
  // 同じチャンクでも引き直すと即返るので、待ち続けずに打ち切る。
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

  const fetchChunk = async (chunk, retry = 2) => {
    let options = {
      method: 'POST',
      body: `terms=${chunk.join(",")}`,
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
      // タイムアウトも 5xx もここに来る。部分的に欠けた木を黙って返さないよう、
      // リトライしても駄目なら中断する
      if (retry > 0) return fetchChunk(chunk, retry - 1);
      throw new Error(`${url} failed (${chunk.length} terms from ${chunk[0]}): ${e.message}`);
    }
  };

  let bindings = [];
  for (let i = 0; i < chunks.length; i += CONCURRENCY) {
    const batch = await Promise.all(chunks.slice(i, i + CONCURRENCY).map(c => fetchChunk(c)));
    batch.forEach(b => { bindings = bindings.concat(b); });
  }

  // 後続の return が SPARQL 結果と同じ形で扱えるようにする
  return {results: {bindings: bindings}};
};
```

## Endpoint

https://rdfportal.org/ncbi/sparql

## `allLeaf`
- 全ヒト NCBI Gene (without annotation 用)
- `typeOfGene` が `biological-region` のものはエンハンサー等で遺伝子ではなく、
  ヒトで 128,261 件あるため除外する。残る 11 種 (protein-coding, ncRNA, pseudo など) は全て含める。
```sparql
PREFIX nuc: <http://ddbj.nig.ac.jp/ontologies/nucleotide/>
PREFIX ncbigene: <https://dbcls.github.io/ncbigene-rdf/ontology.ttl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX taxon: <http://identifiers.org/taxonomy/>
SELECT DISTINCT ?leaf ?leaf_label
FROM <http://rdfportal.org/dataset/ncbigene>
WHERE {
  ?leaf a nuc:Gene ;
        ncbigene:taxid taxon:9606 ;
        ncbigene:typeOfGene ?type ;
        rdfs:label ?leaf_label .
  FILTER (?type != "biological-region")
}
```

## `return`
```javascript
({root, leaf, graph, allLeaf}) => {
  const idPrefix = "http://identifiers.org/ncbigene/";
  const categoryPrefix = "http://purl.obolibrary.org/obo/";
  const withoutId = "unclassified";

  let tree = [
    {
      id: root,
      root: true
    },{
      id: withoutId,
      label: "Unclassified",
      parent: root
    }
  ];

  // 遺伝子 ID -> シンボル。leaf はシンボルを返さないのでここで引けるようにする
  let geneLabel = {};
  allLeaf.results.bindings.forEach(d => {
    geneLabel[d.leaf.value.replace(idPrefix, "")] = d.leaf_label.value;
  })

  let withAnnotation = {};
  // 親子関係
  graph.results.bindings.forEach(d => {
    tree.push({
      id: d.child.value.replace(categoryPrefix, ""),
      label: d.child_label.value,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
    if (d.parent.value.replace(categoryPrefix, "") == root && !tree[0].label)
      tree[0].label = d.parent_label.value; // root の label 挿入
  })
  // アノテーション関係
  leaf.results.bindings.forEach(d => {
    //leaf が root を直接指している場合、evidence code が ND (No biological Data available)
    //unclassfied と区別したほうがよいかもしれないが、とりあえず同じ扱いにしておく
    if (d.parent.value.replace(categoryPrefix, "") == root)
      return;
    const id = d.child.value.replace(idPrefix, "");
    // allLeaf の母集団外 (biological-region や retired) は木に入れない
    if (!geneLabel[id])
      return;
    withAnnotation[id] = true;
    tree.push({
      id: id,
      label: geneLabel[id],
      leaf: true,
      parent: d.parent.value.replace(categoryPrefix, "")
    })
  })
  // アノテーション無し要素
  allLeaf.results.bindings.forEach(d => {
    const id = d.leaf.value.replace(idPrefix, "");
    if (!withAnnotation[id]) {
      tree.push({
        id: id,
        label: d.leaf_label.value,
        leaf: true,
        parent: withoutId
      });
    }
  })

  return tree;	
}
```

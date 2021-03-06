# Pyserini: Reproducing SBERT MS MARCO Results

## Dense Retrieval

Dense retrieval with SBERT, brute-force index:

```bash
$ python -m pyserini.dsearch --topics msmarco-passage-dev-subset \
                             --index msmarco-passage-sbert-bf \
                             --encoded-queries sbert-msmarco-passage-dev-subset \
                             --batch-size 36 \
                             --threads 12 \
                             --output runs/run.msmarco-passage.sbert.bf.tsv \
                             --msmarco
```
> _Optional_: replace `--encoded-queries` by `--encoder sentence-transformers/msmarco-distilbert-base-v3`
> for on-the-fly query encoding.

To evaluate:

```bash
$ python -m pyserini.eval.msmarco_passage_eval msmarco-passage-dev-subset runs/run.msmarco-passage.sbert.bf.tsv
#####################
MRR @10: 0.3313618842952645
QueriesRanked: 6980
#####################
```

We can also use the official TREC evaluation tool `trec_eval` to compute other metrics than MRR@10. 
For that we first need to convert runs and qrels files to the TREC format:

```bash
$ python -m pyserini.eval.convert_msmarco_run_to_trec_run --input runs/run.msmarco-passage.sbert.bf.tsv --output runs/run.msmarco-passage.sbert.bf.trec
$ python -m pyserini.eval.trec_eval -c -mrecall.1000 -mmap msmarco-passage-dev-subset runs/run.msmarco-passage.sbert.bf.trec
map                     all     0.3372
recall_1000             all     0.9558
```

## Hybrid Dense-Sparse Retrieval

Hybrid retrieval with dense-sparse representations (without document expansion):
- dense retrieval with SBERT, brute force index.
- sparse retrieval with BM25 `msmarco-passage` (i.e., default bag-of-words) index.

```bas
$ python -m pyserini.hsearch dense  --index msmarco-passage-sbert-bf \
                                    --encoded-queries sbert-msmarco-passage-dev-subset \
                             sparse --index msmarco-passage \
                             fusion --alpha 0.015  \
                             run    --topics msmarco-passage-dev-subset \
                                    --output runs/run.msmarco-passage.sbert.bf.bm25.tsv \
                                    --batch-size 36 --threads 12 \
                                    --msmarco
```
> _Optional_: replace `--encoded-queries` by `--encoder sentence-transformers/msmarco-distilbert-base-v3`
> for on-the-fly query encoding.

To evaluate:

```bash
$ python -m pyserini.eval.msmarco_passage_eval msmarco-passage-dev-subset runs/run.msmarco-passage.sbert.bf.bm25.tsv
#####################
MRR @10: 0.337881134306635
QueriesRanked: 6980
#####################

$ python -m pyserini.eval.convert_msmarco_run_to_trec_run --input runs/run.msmarco-passage.sbert.bf.bm25.tsv --output runs/run.msmarco-passage.sbert.bf.bm25.trec
$ python -m pyserini.eval.trec_eval -c -mrecall.1000 -mmap msmarco-passage-dev-subset runs/run.msmarco-passage.sbert.bf.bm25.trec
map                     all     0.3445
recall_1000             all     0.9659
```

## Reproduction Log[*](reproducibility.md)

+ Results reproduced by [@lintool](https://github.com/lintool) on 2021-04-02 (commit [`8dcf99`](https://github.com/castorini/pyserini/commit/8dcf99982a7bfd447ce9182ff219a9dad2ddd1f2))

# MinerRAG

**A Trustworthy GeoAI System for Mining Stope Safety**

[📄 Paper (PDF)](./MinerRAG%20-%20A%20Trustworthy%20GeoAI%20System%20for%20Mining%20Stope%20Safety.pdf) ·
[🎬 Demo Video](https://youtu.be/mb9GdCTf5OY) ·
[📚 Citation](#citation)

MinerRAG is a trustworthy GeoAI system for underground mining **stope** safety. It augments
retrieval-augmented generation (RAG) with an incrementally maintained **Stope Knowledge Graph (SKG)**,
so that consultative results are grounded in — and traceable back to — source evidence. MinerRAG has
been deployed in the **Jinchuan Nickel and Cobalt Mine**, one of China's largest underground mining
enterprises.

---

## Motivation

Mining underground natural resources involves complex risks related to the environment, geology, and ore
distribution. Although LLM-based GeoAI opens new opportunities, stope safety decision-making faces two
critical challenges:

1. **Heterogeneous, partial domain data.** Safety knowledge is scattered across business departments
   (mining, equipment, technology) in mixed formats — academic papers (PDF), technical reports (DOC),
   monthly dispatch presentations (PPT), and engineering designs (CAD). Maintaining *stable yet
   experience-aware* domain knowledge over such data is non-trivial.
2. **Hallucination and implicit domain correlations.** 3D longitudinal stopes carry spatial relations
   across panels and levels; operations follow strict temporal orders (excavation must precede
   backfilling); a panel level must be in one of *planned excavation, ongoing excavation, planned
   backfill, ongoing backfill, completed backfill*. Classic RAG cannot exploit these correlations and
   cannot guarantee trustworthy, traceable answers.

## Key Features

- **Unified multi-modal representation.** A standardized pipeline built on
  [Docling](https://github.com/docling-project/docling) converts heterogeneous documents into lightweight
  structural Markdown, preserving heading hierarchy, table layout, and paragraph order — because document
  structure implies semantics.
- **Traceable knowledge updates.** The SKG is maintained top-down and bottom-up through two edge types:
  *factual relation edges* (keeping source segments, evidence text, and canonical relation types) and
  *structural attachment edges* (parent-node profiles fused with Jaro–Winkler similarity into a routing
  score for automatic insertion, expert review, or temporary deferral).
- **Correlation-aware consultation.** Retrieval and generation are organized around three hidden
  correlations rather than plain top-k chunks, and every answer can be exported as an archival report
  with traceable citations.

## System Architecture

MinerRAG is organized into three layers.

### 1. Data Processing Layer

- **Standardized pipeline:** document parsing → OCR & text extraction → structure preservation →
  structured enhancement, producing unified Markdown.
- **Chunking & indexing:** semantic chunking with automatic metadata tagging (source file name, page
  number) for filtering and traceability; domain-specific **BGE-m3** embeddings indexed in **Chroma**.
- **SKG maintenance:** domain experts define the business-oriented skeleton (mining operations,
  equipment systems, technical management); new knowledge is attached as factual relation edges or
  structural attachment edges under domain constraints.

### 2. Retrieval Layer

- **Local Retrieval** — *Vector Recall* matches queries against document chunks and traces each chunk
  back to its source page; *Graph Recall* translates natural-language queries into graph queries
  (**Text2Cypher**) to traverse the SKG and recover entities, relations, and reasoning paths. Evidence is
  fused in an evidence-first order, and candidates retain their provenance before generation.
- **Web Retrieval** — evaluates query coverage and integrity; when local evidence is insufficient, the
  query is decomposed into sub-questions and answered from trusted external sources. Retrieved
  information is added as a *supplement*, never as a replacement for local evidence.

### 3. Consulting Layer

Three consultative entries, each designed around one hidden correlation:

| Entry | Correlation | Behavior |
| --- | --- | --- |
| **3D Entry** | Spatiotemporal | Binds a selected stope to panel-level topology and operational states; jointly analyzes neighboring stopes, current operation states, and weather before consulting. |
| **Document Entry** | Specific Knowledge | Restricts the consultation to given regulations, engineering reports, or geological investigations; prioritizes local retrieval and uses web retrieval only as a supplement. |
| **Graph Entry** | Knowledge Topology | Exposes the five-layer topology (*mine area – category – domain – instance – keyword*) and returns a traceable subgraph anchored to explicit entities, relations, and source evidence. |

## Evaluation

MinerRAG's SKG update component, **MinerRAG-KG**, is evaluated on a manually annotated subset of
Jinchuan No. 2 area: 14 papers, 728 text segments, 585 candidate factual relation segments, and 60 valid
factual relation edges; 117 entities are split into 35 validation and 82 test samples.

| Task | Method | Metric | Score | Src./Evid. |
| --- | --- | --- | ---: | :---: |
| Factual edge | Rule-Pattern | Precision | 8.76% | 100% / 100% |
| Factual edge | LLM-Direct | Precision | 22.76% | 0% / 0% |
| Factual edge | **MinerRAG-KG** | Precision | **26.47%** | **100% / 100%** |
| Structural attach. | NameSim | Top-1 Acc. | 36.59% | – |
| Structural attach. | LLM-Direct | Top-1 Acc. | 68.29% | – |
| Structural attach. | **MinerRAG-KG** | Top-1 Acc. | **78.05%** | – |

The comparison is deliberately **auditability-oriented**. On the full KG update task, LLM-Direct attains
higher recall (46.67% vs. 30.00%), but MinerRAG-KG filters low-quality candidates before insertion and
keeps complete source/evidence for every edge. With the Jaro–Winkler routing signal (α = 0.50,
T_a = 0.80, T_r = 0.75) selected on the validation set, automatic structural attachment reaches 86.96%
precision and intercepts 83.33% of incorrect attachments before graph insertion.

A case study on Jinchuan No. 2 area shows the separation between candidate generation and graph
insertion: one newly added standard document yields 19 candidate knowledge items, and after routing and
manual verification the update results in 7 newly created nodes, 19 merged nodes, 13 added relations, and
3 discarded relations.

## Deployment

- **Site:** Jinchuan Nickel and Cobalt Mine, China
- **Data:** 10,000+ files — 8,000+ academic papers, 3,000+ technical reports, 200+ monthly dispatch
  presentations, 1,000+ engineering designs
- **Knowledge:** 1,156 entities and 1,184 relations in the SKG
- **LLM backbones:** configurable across DeepSeek-V4-Pro, Qwen 3.5, and ChatGLM 4.7
- **Daily use:** stope safety consultation, plus export of archival PDF reports (title, summary, and
  logically connected sections) with traceable citation sources

## Citation

```bibtex
@inproceedings{tong2026minerrag,
  title     = {MinerRAG: A Trustworthy GeoAI System for Mining Stope Safety},
  author    = {Tong, Shuaijie and Ding, Weilong and Yang, Xinyu},
  booktitle = {IEEE International Conference on Data Mining (ICDM)},
  year      = {2026}
}
```

> Update `pages` and `doi` once the proceedings are published.

## Contact

<tsjkht@foxmail.com>

## Acknowledgment

This work was supported by the Key Project of Graduate Education and Teaching Reform of North China
University of Technology.

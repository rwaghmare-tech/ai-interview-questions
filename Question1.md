# Question 1: Your RAG system works perfectly on plain text PDFs but fails badly on scanned PDFs. Why?

> Hint: OCR quality, layout, missing structure

---

# Core Concept

The core issue is that scanned PDFs are fundamentally different from text-based PDFs.

A text PDF already contains machine-readable text layers, so extraction is deterministic and structurally clean.

But scanned PDFs are essentially images, so the RAG pipeline first depends on OCR before retrieval even begins.

So the failure is usually not in:
- the LLM
- the vector database
- embeddings

The failure starts during **document ingestion**.

---

# Why Scanned PDFs Fail

Scanned PDFs introduce multiple problems:

## 1. OCR Errors

OCR reconstructs text probabilistically from pixels.

Example:

Original:
```text
Policy Number
```

OCR Output:
```text
Po1icy Nurnber
```

Even small OCR mistakes change token distributions and embedding vectors.

Another example:

Original:
```text
$5000
```

OCR:
```text
$500O
```

Here:
- `0` becomes `O`
- semantic meaning changes
- retrieval quality drops

---

## 2. Layout Loss

OCR systems may extract content in the wrong order.

Example:

Correct Table:
```text
Employee Name | Salary
Rahul         | 120000
```

Broken OCR:
```text
Employee
Rahul
Name
120000
Salary
```

Now:
- table relationships are lost
- chunking becomes unreliable
- retrieval quality degrades

---

## 3. Missing Structural Information

Scanned PDFs lose important document signals such as:
- headings
- tables
- hierarchy
- reading order
- figure references
- column relationships

The retriever receives flattened noisy text instead of structured semantic content.

---

# Why Retrieval Quality Drops

Embedding models assume semantic continuity.

OCR corruption changes tokens and therefore changes embedding vectors.

Example:

Original:
```text
Quarterly revenue increased by 18%
```

OCR Output:
```text
Quarter1y revenue increased by 1B%
```

Now the embedding vector moves away from the correct semantic neighborhood.

This causes:
- relevant chunks to fall below top-k retrieval
- irrelevant chunks to appear more similar

---

# Mathematical Intuition

Most vector databases use cosine similarity:

$$
\cos(\theta) = \frac{A \cdot B}{||A|| ||B||}
$$

Where:
- \(A\) = query embedding
- \(B\) = document embedding

OCR corruption changes the direction of vector \(B\).

Even small token errors can reduce cosine similarity enough to break retrieval.

---

# Important Insight

A critical production insight is:

> Character-level OCR accuracy does NOT guarantee retrieval accuracy.

Even:
- 95% OCR accuracy
- low token error rate

can still completely break:
- tables
- financial statements
- semantic relationships
- document structure

So:

$$
\text{RAG Quality}
\propto
\text{Extraction Quality}
\times
\text{Retrieval Quality}
\times
\text{Generation Quality}
$$

Poor extraction quality propagates through the entire pipeline.

---

# Real Production Example

Suppose an insurance company tests its RAG system using:
- digitally generated PDFs
- clean text documents

Everything works perfectly.

But in production:
- users upload scanned claim forms
- handwritten notes appear
- tables become malformed
- claim IDs are corrupted

The team may incorrectly blame:
- embeddings
- chunking
- prompting

But the real issue is poor OCR and layout extraction.

This is extremely common in enterprise GenAI systems.

---

# Advanced Failure Mode

Even high OCR accuracy may still fail.

Why?

Because RAG depends not only on text extraction, but also on:
- semantic integrity
- contextual continuity
- layout preservation

A table with incorrect row alignment may still have high OCR accuracy while being semantically unusable.

---

# Mitigation Strategies

## 1. Use Better OCR Systems

Instead of generic OCR, use:
- PaddleOCR
- Azure Document Intelligence
- Google Document AI
- AWS Textract

These preserve structure more effectively.

---

## 2. Use Layout-Aware Models

Use models that preserve:
- reading order
- tables
- coordinates
- sections

Examples:
- LayoutLM
- Donut
- Nougat

---

## 3. Multi-Modal RAG

For:
- diagrams
- charts
- images

Use:
- vision-language models
- image captioning
- multimodal embeddings

---

## 4. Confidence-Aware Pipelines

Store OCR confidence scores.

If confidence is low:
- trigger human review
- rerun extraction
- avoid indexing corrupted chunks

---

# Interview-Ready Closing Statement

In production RAG systems, retrieval quality is often limited more by document ingestion and structure preservation than by the LLM itself.

Scanned PDFs expose this weakness because OCR introduces semantic and structural noise before retrieval even begins.

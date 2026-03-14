# 🔐 Data Privacy & Anonymization

![Status: Complete](https://img.shields.io/badge/Status-Complete-success?style=flat-square)
![Topic: Data Privacy](https://img.shields.io/badge/Topic-Data%20Privacy-blueviolet?style=flat-square)

This directory contains comprehensive notes, architectural diagrams, and mathematical proofs exploring the paradigms of data privacy. It focuses on the balance between maintaining the utility of statistical databases and mathematically guaranteeing the anonymity of the individuals within them.

> ✅ **Project Status: Complete** > The notes and diagrams in this module are finalized and serve as a complete reference guide for core data privacy concepts.

## 📖 Content Summary

The documentation in this folder covers both the theoretical threats to privacy and the formal mechanisms used to defend against them:

* **The Threat of Re-identification:** Understanding how supposedly "anonymized" data can be unmasked.
* **Quasi-Identifiers:** Analysis of how combined attributes (like ZIP code, gender, and birthdate) can uniquely single out individuals in a dataset.
* **Linkage Attacks:** The mechanics of cross-referencing multiple public or leaked datasets to extract sensitive information.
* **k-Anonymity:** Structural models and techniques for masking data to ensure each record is indistinguishable from at least *k-1* other records.
* **Differential Privacy:** Formal frameworks for quantifying privacy loss, including proofs for the Laplace mechanism, global sensitivity, and sequential composition.

## 🖼️ Included Diagrams

To supplement the markdown notes, this directory includes several visual aids to demonstrate complex attacks and data structures:
* `Linkage attack.png` - Visualizes the flow of cross-referencing datasets.
* `The variation of quasi-identifier.png` - Demonstrates how attribute combinations reduce anonymity sets.
* `Threat of re-identification.png` - Maps the pipeline of deanonymizing data.

---
*These documents were compiled through academic coursework, independent research, and practical application in computer systems engineering.*
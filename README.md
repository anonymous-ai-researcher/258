<h1 align="center">Forget, but Not Lost</h1>
<h3 align="center">Strong Forgetting for Expressive Ontologies</h3>

<p align="center">
  <img src="https://img.shields.io/badge/Logic-ALCQI(∇)-blue" alt="Logic">
  <img src="https://img.shields.io/badge/Forgetting-Strong-purple" alt="Strong Forgetting">
  <img src="https://img.shields.io/badge/Language-Java-orange" alt="Java">
  <img src="https://img.shields.io/badge/JDK-1.8-red" alt="JDK">
  <img src="https://img.shields.io/badge/License-MIT-green" alt="License">
</p>

---

## 📄 Extended Version with Full Proofs

> **The extended version of this paper, including all omitted proofs, is available in this repository:**
> 
> 📥 **[`extended-version.pdf`](./extended-version.pdf)**
> 
> This document contains the complete proofs of all lemmas and theorems, including the soundness and completeness of the unified inference rule **IR**, the normalization correctness lemmas, and the termination argument with the double-exponential bound.

---

## 🔍 Overview

This repository provides the implementation, experimental data, and extended materials for our paper on **strong role forgetting** in $\mathcal{ALCQI}(\nabla)$ ontologies, the first practical method to handle the full interplay between **qualified number restrictions** ($\mathcal{Q}$) and **inverse roles** ($\mathcal{I}$).

**Why this is hard:** In $\mathcal{ALCQ}$ (no inverse roles), constraints propagate in a single direction along the role structure. Introducing inverse roles creates *bidirectional* dependencies that destroy the tree model property and require reasoning about four distinct types of arithmetic interactions:

| Interaction | Literals | Status |
| --- | --- | --- |
| **Forward vs. Forward** | ${\geq}{}{r}.$ vs. ${\leq}{}{r}.$ | Classical |
| **Inverse vs. Inverse** | ${\geq}{}{r^-}.$ vs. ${\leq}{}{r^-}.$ | Classical |
| **Forward vs. Inverse** | ${\geq}{}{r}.$ vs. ${\geq}{}{r^-}.$ | **Novel** |
| **Inverse vs. Forward** | ${\leq}{}{r^-}.$ vs. ${\leq}{}{r}.$ | **Novel** |

Our unified inference rule **IR** resolves all four cases through purely syntactic inference, introducing the universal role $\nabla$ when the result falls outside $\mathcal{ALCQI}$.

---

## 📁 Repository Structure

```
.
├── 📄 extended-version.pdf        # Full paper with all proofs
├── 📂 src/                        # Java source code
│   ├── concepts/                  # Concept representations
│   ├── roles/                     # Role representations (incl. inverse)
│   ├── convertion/                # OWL ↔ internal format conversion
│   ├── forgetting/                # Core role-forgetting calculus (IR rule)
│   ├── formula/                   # Formula data structures
│   └── evaluation/                # Evaluation scripts & demo
├── 📂 Dependency/                 # Required JAR libraries
├── 📂 test_forgetting.zip         # Benchmark datasets (Oxford-ISG + BioPortal)
└── 📄 README.md
```

---

## ⚙️ Environment Requirements

| Requirement | Version |
| --- | --- |
| **JDK** | 1.8 |
| **IDE** | IntelliJ IDEA (recommended) |
| **Dependencies** | All JARs in `Dependency/` folder |

---

## 🚀 Getting Started

### 1. Setup

Unzip the entire project.

### 2. Import into IntelliJ IDEA

1. Open IDEA → **File** → **Open** → select the project folder
2. Right-click `src/` → **Mark Directory as** → **Sources Root**
3. **File** → **Project Structure** → **Libraries** → **+** → add all JARs from `Dependency/`

### 3. Prepare Datasets

The **Oxford-ISG** and **BioPortal** benchmarks are in the compressed package **`test_forgetting.zip`**. Unzip it before running experiments.

---

## 💻 Usage

### Computing Strong Role Forgetting

The code computes the result of **strong role forgetting** for $\mathcal{ALCQI}(\nabla)$ ontologies. The following template shows how to compute a forgetting result:

```java
package evaluation;

import concepts.AtomicConcept;
import convertion.BackConverter;
import convertion.Converter;
import forgetting.Fame;
import formula.Formula;
import org.semanticweb.owlapi.apibinding.OWLManager;
import org.semanticweb.owlapi.io.IRIDocumentSource;
import org.semanticweb.owlapi.io.OWLXMLOntologyFormat;
import org.semanticweb.owlapi.model.*;
import roles.AtomicRole;

import java.io.*;
import java.util.*;

public class Demo {

    public static void main(String[] args) throws Exception {

        OWLOntologyManager manager = OWLManager.createOWLOntologyManager();

        // ── Input / Output paths ──────────────────────────
        String filePath   = "path/to/input-ontology.owl";   // TODO: target ontology
        String outputPath = "path/to/output-result.owl";    // TODO: save path

        // ── Load ontology ─────────────────────────────────
        File file = new File(filePath);
        IRI iri = IRI.create(file);
        OWLOntology ontology = manager.loadOntologyFromOntologyDocument(
            new IRIDocumentSource(iri),
            new OWLOntologyLoaderConfiguration().setLoadAnnotationAxioms(true)
        );

        // ── Convert to internal representation ────────────
        Converter converter = new Converter();
        converter.CReset();
        Fame fame = new Fame();

        // roleList contains all role names occurring in the input ontology.
        List<AtomicRole> roleList =
            converter.getRolesInSignature_ShortForm(ontology);

        // ── Select role names to forget ───────────────────
        // TODO: Implement a selection strategy, e.g.
        //       Set<AtomicRole> r_sig = select(roleList);
        Set<AtomicRole> r_sig = null;

        // ── Run role forgetting ───────────────────────────
        List<Formula> formulaList =
            converter.OntologyConverter_ShortForm(ontology);
        List<Formula> resultList =
            fame.FameCR(r_sig, formulaList);

        // ── Convert back and save ─────────────────────────
        BackConverter backConverter = new BackConverter();
        OWLOntology ontoSol = backConverter.toOWLOntology(resultList);

        File outFile = new File(outputPath);
        OutputStream os = new FileOutputStream(outFile);
        manager.saveOntology(ontoSol, new OWLXMLOntologyFormat(), os);

        System.out.println("Forgetting result saved to: " + outputPath);
    }
}
```

> **Note.** Role forgetting subsumes concept forgetting: any concept name $A$ can be eliminated by introducing a fresh role $r$, replacing $A$ with $\exists r.\top$, and then forgetting $r$.

---

## 📊 Experimental Results (Summary)

Evaluated on **338 real-world ontologies** from two benchmark repositories:

| Metric | Result |
| --- | --- |
| **Success rate** | up to 90% |
| **Speedup vs. state of the art** | 3.6–4.8× faster |
| **Memory** | Significantly lower |
| **Logic supported** | $\mathcal{ALCQI}(\nabla)$ — first to handle $\mathcal{Q}$ + $\mathcal{I}$ together |

---

## 📖 Citation

```
Paper under review.
```

---

<p align="center"><i>Anonymous submission</i></p>

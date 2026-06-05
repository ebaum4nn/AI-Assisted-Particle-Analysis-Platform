# AI-Assisted Microscopy Particle Analysis Platform

**An auditable, end-to-end pipeline for detecting, measuring, and reporting on particles in microscopy imagery — built for environments where every result has to be explainable.**

[Demo Video Page on BaumannLabs.com](https://baumannlabs.com/ai-assisted-microscopy-particle-analysis-platform/) — a full run from image upload through detection, analyst review, and a generated report.

---

## The problem

Microscopy particle analysis — counting and sizing cells, microplastics, or other particulates — is slow, manual, and hard to reproduce. The harder problem isn't detection accuracy; it's **trust**. In regulated and diagnostic settings, an automated result is worthless unless you can answer: *which model version produced this, on what input, with what settings, and who signed off?*

I built this platform to treat that question as a first-class design constraint rather than an afterthought.

## What it does

An analyst uploads a microscopy image and gets back a reviewed, reportable result:

```
Upload → Preprocess → [Optional Denoise] → Detect & Segment → Analyst Review → Report
```

- **Detection & segmentation** — locates individual particles, draws masks, and classifies them by type
- **Real-world measurement** — converts pixel measurements to microns using per-job calibration, producing size distributions and per-class statistics
- **Optional generative denoising** — an edge-preserving cleanup pass for noisy inputs, deliberately constrained so it never invents structure that wasn't there
- **Human-in-the-loop review** — a reviewer can adjust confidence thresholds, inspect every detection, and approve / reject / flag with structured reason codes before anything is finalized
- **One-click reporting** — a generated PDF with annotated imagery, size histograms, summary statistics, and a written summary

## Why it's built the way it is

The interesting engineering is in the infrastructure underneath the detection model. Three design decisions drove everything:

### 1. Reproducibility is enforced, not hoped for
Every job records the exact pipeline configuration and the exact model version used, fingerprinted so that any result can be tied back to the precise setup that produced it. Re-running the same input under the same configuration is meant to give you the same answer — and you can *prove* which configuration that was.

### 2. The audit trail is immutable
Every processing step writes a permanent, append-only record: what ran, which versioned component did it, how long it took, and integrity fingerprints of its input and output. Records are never updated or deleted. When an analyst makes a correction, that correction is stored *separately* rather than overwriting history — so the original machine output and the human judgment are both preserved. This is the foundation a real compliance pathway would build on.

### 3. The pipeline is configuration, not code
Each processing stage is an independent, versioned, hot-swappable component with typed input/output contracts. The full pipeline — which stages run, in what order, with what settings — is defined declaratively. Adding a new analysis type or reordering the flow doesn't require touching the engine; the contracts between stages keep everything type-safe. The same underlying platform is designed to host multiple analysis products, not just this one.

## The review loop

A detection result is a starting point, not a verdict. The review interface lets an analyst:

- Slide the confidence threshold and watch statistics update live
- Inspect a before/after view when denoising was applied
- Approve, reject, or flag individual detections with structured reason codes
- Capture a final decision with reviewer credentials attached

Every review is linked back to the exact model and pipeline version it judged — closing the loop between automated output and human accountability, and laying the groundwork for a future retraining feedback cycle.

## Technical stack

| Layer | Technology |
|-------|-----------|
| **Detection / Segmentation** | Deep-learning instance segmentation with per-tile inference and cross-tile reconciliation for large images |
| **Generative denoising** | Diffusion-based, edge-guided, hard-constrained against hallucination; GPU inference via serverless workers |
| **Backend / API** | Python, FastAPI, async job processing, real-time status streaming |
| **Pipeline engine** | Custom plugin architecture with typed data contracts and a versioned model registry |
| **Queue / workers** | Distributed task queue with live stage-by-stage event streaming |
| **Data layer** | PostgreSQL with versioned, migration-managed schema |
| **Frontend** | Next.js / React — upload, live processing timeline, interactive review workspace |
| **Reporting** | Programmatic PDF generation with embedded charts and annotated imagery |
| **Infrastructure** | Containerized (Docker Compose) local stack; GPU inference offloaded to on-demand cloud workers |

## What the demo shows

1. **Upload & calibration** — an image goes in with its micron-per-pixel calibration
2. **Live processing** — each stage reports in real time as it runs, with an audit entry appearing for every step
3. **Review** — detections rendered over the image, an interactive confidence threshold, and live-updating statistics
4. **Report** — the generated PDF with annotated imagery, size distributions, and summary stats

## Engineering takeaways

This project is where I brought together the parts of my work I care most about: production AI systems that are **measurable, reproducible, and accountable**. The model is one component — the harder and more valuable work was the platform around it that makes its outputs trustworthy enough to put in front of someone making a real decision.

Built end-to-end: architecture, detection pipeline, the auditable platform layer beneath it, the backend and job system, the review frontend, and the reporting.

---

*Implementation details, source, and model specifics are intentionally omitted. Happy to walk through the architecture and engineering decisions in depth on request.*

**Eric Baumann** · [baumannlabs.com](https://baumannlabs.com) · [linkedin.com/in/ericbaumann1](https://linkedin.com/in/ericbaumann1)

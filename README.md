# An OCR engine for a historical military archive

**Two GPUs, five readers and not a single invented word.**
Preliminary technical report (19 pages, English and Spanish). Work in progress.

> Status: the engine is built and has been tested on about 4,000 real pages. The main work is still ahead: the archive's 300,000+ pages. Code and scans are not released.

## The problem

The [Museo de Historia Militar](https://museohistoriamilitar.com) in Málaga is a private collection devoted to Spanish military uniforms. It is closed to the public while I look for new premises, so I am digitising its uniform archive to put the documentation and the catalogue online in the meantime: more than 300,000 pages of regulations, booklets, official gazettes, army lists, plates and handwritten documents from the 18th to the 20th century.

![One shelf of the archive: bound volumes and binders by year](figures/archive_shelf.jpg)

One requirement drives the whole design: **the system must not invent anything.** Plausible but false text is worse than a gap, because it enters the catalogue and nobody looks at it again. Whatever is not read with certainty is returned as `[ilegible]` together with the candidate readings, and it stays that way until two independent readers agree.

![A real gap, before and after](figures/fig03_example_p67.png)

## How it works

Everything runs on my own hardware, with a cloud model only as a last resort.

- **Stage 1, server with an RTX 5090 (32 GB):** PaddleOCR reads lines with boxes and confidence; glm-ocr gives a second reading of tables and doubtful lines; qwen3.8 classifies the document and reads figures and captions. A word-by-word comparison marks every mismatch as `[ilegible]` with its candidates.
- **Stage 2, second machine with an RTX 5070 Ti (16 GB):** gemma4, a model from a different family, re-reads the gaps and votes. It runs in parallel with the next document.
- **Last resort:** Gemini 3.8 Flash sees the whole page and the position of each remaining gap. It also transcribes handwritten documents directly.
- **Tables:** two readings plus arithmetic. Row and column sums are the only evidence that does not depend on any model.
- **Dictionary and human review:** what remains `[ilegible]` goes to a person with the crop and the readings; confirmed words vote from then on.

![The whole system, from input to catalogue](figures/fig01_system.png)

## Results so far

| Measurement | Value |
|---|---|
| One production night (about 8 hours, version 1.4) | 4,093 pages, about 6.8 s per page |
| Lines read that night | 511,017; 96 % with no doubt |
| Table cells corrected by sum checks | 505; 225 of 1,757 tables left flagged for review |
| Batch of eight documents, single stage vs chained stages | 451 s vs 318 s |
| `[ilegible]` marks resolved, local models only vs with Gemini | 46-49 % vs 87-89 % |

![Share of ilegible marks resolved](figures/fig10_resolved.png)

## What worked and what did not

Three lessons so far: marking instead of guessing prevents errors nobody would look at again; a reader from another model family catches errors that two engines share; and table sums are the only objective check. Almost everything that failed had the same root: asking a model to read with nothing to check it against, or paying a fixed cost on documents that did not need it.

![What worked and what did not](figures/fig12_worked_not.png)

## What is not measured yet

This is a preliminary report and it says so where it matters:

- There is no hand-checked reference set yet, so the figures say how much the system doubts and how much it resolves, not how many errors remain unmarked.
- The document classifier is not validated.
- Gemini's confidence is self-reported, not calibrated, and its cost is not measured at scale.

The next step is a block of about 10,000 pages with a 200-page reference set, then the full archive. A second report will follow.

## Files

- `report/archive_ocr_engine_report_public.pdf` — the report in English
- `report/informe_ocr_archivo_uniformidad_publico.pdf` — the report in Spanish
- `figures/` — the twelve figures of the report and a photo of the archive

## Author

Javier Chaos — software architect / tech lead (.NET, Azure, Python, computer vision and applied AI).
[LinkedIn](https://www.linkedin.com/in/javierchaos/) · [GitHub](https://github.com/jachaos)

## Licence

Report, text and figures: CC BY 4.0.

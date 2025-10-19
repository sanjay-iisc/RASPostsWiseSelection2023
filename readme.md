# RAS 2023 — Rank-wise Seat Allocation (Unofficial) 📊

> **⚠️ Important disclaimer**
>
> This project provides an **unofficial**, **approximate** rank-wise check for RAS 2023 seat allocation.  
> It is **not** the final result. I may have made mistakes. Treat this as a *rough indication only*.

---

## What this is

A small, reproducible analysis to help you **check—rank by rank—whether you’re likely to get a post in RAS 2023**.  
It combines:

- The **RAS 2023 results** (recently published)  
- A **seat matrix** per **post** (POST-1 … POST-33) and **category/sub-category**

Because I don’t have everyone’s **post preferences**, the simulation simply allocates from **POST-1 (SDM)** through **POST-33 (Programmer)** *in order*, for both **main** and **subordinate** seats, after **removing seats reserved for SA** candidates.

---

## Key assumptions (read carefully)

1. **Preference-blind allocation:** Candidates are processed by rank; posts are considered in fixed order **POST-1 → POST-33** (SDM → Programmer).  
2. **Removed:** Seats **reserved for SA** candidates are **excluded** from the available pool.  
3. **Excluded categories:** **PH**, **NG**, **DC**, **SP** are **not** considered as separate buckets.
   - If a person belongs to one of these, I fall back to their **relevant base category** (e.g., **SC, NG → SC** if applicable).
4. **Heuristic buffer:** Because of the simplifications above, please mentally apply **“-10 ranks”** as a relaxtion margin due to assumption 3 to your result.
5. **No finality:** Results are **indicative only**—real allocation may differ due to preferences, tie-breaks, horizontal reservations, legal orders, etc.

---

## Repository structure

```
.
├── main.ipynb               # Notebook to run the full pipeline end-to-end
├── sheetMatrix.xlsx         # Seat matrix (posts × categories/sub-categories)

│── RAS_2023_results.pdf # Source results (or CSV if you have it)

── README.md                # You are here
```

> If you see `/sheatMatrix.xlsx` mentioned in older notes, that’s the same file as `sheetMatrix.xlsx`.

---

## Quick start

### 1) Requirements

- Python **3.9+**
- Jupyter (for `.ipynb`), or run scripts directly
- Python libraries:
  - `pdfplumber`
  - `pandas`
  - (optional) `numpy`, `tqdm`, `openpyxl`

### 2) Install

```bash
# create & activate a virtual environment (recommended)
python3 -m venv .venv
source .venv/bin/activate   # on Windows: .venv\Scripts\activate

# install libs
pip install -U pip
pip install pdfplumber pandas numpy tqdm openpyxl jupyter
```

### 3) Run

**Option A — Notebook**

```bash
jupyter notebook
```

Open `main.ipynb` and execute all cells.

**Option B — Scripts (example)**

```bash
python -m src.parse_pdf data/RAS_2023_results.pdf -o data/results.csv
python -m src.normalize data/results.csv -o data/results_clean.csv
python -m src.allocate data/results_clean.csv sheetMatrix.xlsx -o out/allocation_report.xlsx
```

Outputs (typical):

- `out/allocation_report.xlsx` — per-rank allocation trace
- `out/summary.csv` — seats remaining per post/category after each step
- `out/logs.txt` — decisions and fallbacks

---

## How the allocation works (high level)

1. **Parse results**: Extract roll no., rank, and category tokens from the RAS 2023 results (PDF/CSV).  
2. **Normalize categories**: Tokenize and standardize labels (e.g., `GE,WE`, `BC,NG,RG`), and map **PH/NG/DC/SP** to the **base category** where needed.  
3. **Load seat matrix**: `sheetMatrix.xlsx` with a **(Post × Category/Subcategory)** grid. Remove **SA-reserved** seats.  
4. **Simulate**:
   - Iterate candidates by **increasing rank**
   - For each candidate, traverse the post list **POST-1 → POST-33**, decrementing the first compatible seat bucket found
   - Record the **assigned post**, **category bucket** used, and **remaining seats**
5. **Export** results and summaries.

---

## Interpreting your result

- If the report says you’re **allocated** a post at your rank: that’s the **simulated** outcome **under the assumptions above**.  
- Because this ignores true **preferences** and some **horizontal/complex** rules, consider a **±10 rank** buffer to gauge safety.  
- **Not allocated** in the simulation doesn’t mean “no chance”—it may change when preferences and official rules are applied.

---

## Customizing the run

- **Change the +10 buffer**: This is just guidance; the code itself does not add +10. Adjust your interpretation or add a sensitivity test in the notebook.  
- **Edit seat matrix**: Update `sheetMatrix.xlsx` if you have revised official numbers (e.g., different SA removals).  
- **Include excluded categories**: If you want to model **PH/NG/DC/SP** explicitly, extend the mapping in `normalize.py` and add columns in `sheetMatrix.xlsx`.  
- **Post order**: If you want to simulate a different order or candidate preferences, modify the post traversal in `allocate.py`.

---

## Known limitations

- **No preference modeling** (biggest source of deviation).  
- **Simplified category fallback** for **PH/NG/DC/SP** (treated under base category).  
- **SA seats removed** globally based on the provided matrix (verify with official data).  
- **Parsing noise** if the PDF layout changes (tune `pdfplumber` thresholds).

---

## Reproducibility checklist

- ✅ Python version and packages pinned (see above)  
- ✅ Inputs (`sheetMatrix.xlsx`, `RAS_2023_results.pdf/CSV`) placed as documented  
- ✅ Run `main.ipynb` top-to-bottom without manual edits  
- ✅ Commit seed/random choices if you add any stochastic components (currently none)

---

## FAQ

**Q: Where do I see how many seats are left per post/category as allocation proceeds?**  
See `out/summary.csv` and the “Remaining seats” table in the notebook.

**Q: I belong to NG/DC/SP/PH—how were we handled?**  
In this version, those are **not** modeled as distinct buckets; they’re mapped to your **base category** (e.g., NG under SC if applicable). You can extend the model to handle them explicitly.

**Q: The PDF isn’t parsing correctly.**  
Adjust the tolerance parameters in `src/parse_pdf.py` (`y_tolerance`, word grouping) or try a CSV source.

---

## Contributing / Reporting mistakes

If you spot a mistake or have official corrections (especially for **seat matrix** or **SA removals**), please open an issue or comment.  
Suggestions and PRs to add **preference modeling** or **horizontal reservations** are very welcome.

---

## License

This project is shared for public benefit and transparency. Use at your own risk.  
Please credit the repository if you reuse code or data.

---

## A final word

This tool is meant to **reduce uncertainty**, not add to it.  
**Best of luck** for your posting—and if something looks off, **please let me know in the comments so I can fix it.** 🙏
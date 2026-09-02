# Legal Metrology (Packaged Commodities) Rules, 2011 Compliance Inspector

A full-stack compliance inspection application for packaged commodities in India under the **Legal Metrology (Packaged Commodities) Rules, 2011** (enacted under the Legal Metrology Act, 2009).

The system extracts text and spatial bounding boxes from packaged commodity label photos, classifies the statutory declarations, validates them against a **deterministic, versioned rules engine**, renders visual pass/fail compliance audits with exact legal citations, and exports audit-ready PDF reports.

---

## 1. Key Capabilities & Scope (Phase 1 MVP)

- **Label Image Ingestion & Pre-Processing**:
  - Drag-and-drop, file upload, or camera capture.
  - Built-in OpenCV preprocessing pipeline (deskewing, CLAHE contrast enhancement, glare & noise reduction).
  - Bundled standard verification samples (Compliant FMCG Snack, Non-Compliant MRP Tax, Banned Quantity Words) for 1-click test driving.
- **Statutory Applicability Pre-Check (Section 6.4)**:
  - Gating checks for Rule 3 exemptions (packages > 25 kg / 25 L, or goods for industrial/institutional consumers).
  - Gating checks for Rule 26(a) exemptions (net quantity ≤ 10g / 10ml).
  - Commodity-specific exemptions (e.g., agarbatti/bidis exempt from month/year under Rule 6(1)(g) proviso A).
- **OCR with Spatial Bounding Boxes**:
  - Extracts text and computes normalized bounding box coordinates (`x, y, w, h` and polygons).
  - Estimates numeral heights in pixels and millimeters.
- **Interactive OCR Review & Inline Editor**:
  - Side-by-side interactive split view: Label image with SVG bounding box overlays on the left, editable monospace text cards on the right.
  - Low-friction inline text editing and statutory category reassignment.
- **Deterministic, Versioned Rules Engine**:
  - Configured via `backend/rules_config/rules_v2011.json`.
  - Validates all 7 Chapter II mandatory declarations:
    - **Rule 6(1)(a)**: Manufacturer / Packer / Importer name & address with required qualifier (`"Mfg by"`, `"Packed by"`, `"Marketed by"`).
    - **Rule 6(1)(b)**: Common / Generic name of the commodity.
    - **Rule 6(1)(c), Rule 12 & 13**: Net quantity in standard SI units (`g`, `kg`, `ml`, `l`, `cm`, `m`, `N`), unit-scale validation, prohibited qualifier detection (`"approx"`, `"minimum"`, `"not less than"`), and banned non-SI terms (`"dozen"`, `"gross"`).
    - **Rule 6(1)(d)**: Month & year of manufacture/packing/import.
    - **Rule 6(1)(e), Rule 2(m), Rule 18**: Retail sale price (MRP) format check (`"MRP Rs.___ incl. of all taxes"` / `"Maximum retail price Rs.___ inclusive of all taxes"`), currency symbol check (₹/Rs.), and soft paise rounding check.
    - **Rule 6(1)(f)**: Dimensions declaration (conditional for size-relevant goods like textiles, cables).
    - **Rule 6(2)**: Consumer care details (helpline phone number, email address, postal address).
    - **Rule 7**: Numeral height verification (transparently flagged as unverified without reference scale).
    - **Rule 9(4)**: Language & script check (Hindi in Devanagari or English in Latin).
- **Audit-Ready PDF Report Export**:
  - Publication-quality official compliance report generated with ReportLab.
  - Features overall verdict banner, product metadata, label snapshot, and itemized statutory checklist table with legal citations.
- **Inspection Audit History**:
  - Persistent SQLite / PostgreSQL database recording all scans, timestamps, product names, and pass/fail statuses.

---

## 2. Architecture & Tech Stack

```
c:\v4\
├── backend\
│   ├── rules_config\
│   │   └── rules_v2011.json            # Versioned JSON statutory rules config
│   ├── rules_engine\
│   │   ├── engine.py                   # Deterministic rule evaluator
│   │   ├── exemptions.py               # Rule 3, 26, and proviso exemptions
│   │   ├── declaration_rules.py        # Rule 6(1)(a)-(f), Rule 6(2) mandatory checks
│   │   ├── mrp_rules.py                # Rule 2(m), Rule 6(1)(e), Rule 18 format & tax checks
│   │   ├── net_quantity_rules.py       # Rule 12, 13 SI units, scale, & banned terms check
│   │   ├── font_language_rules.py      # Rule 7 height estimation & Rule 9(4) language check
│   │   └── schemas.py                  # Pydantic schemas
│   ├── ocr\
│   │   ├── preprocessor.py             # OpenCV deskew, CLAHE contrast, glare reduction
│   │   ├── extractor.py                # EasyOCR / OCR pipeline with bounding boxes & DPI
│   │   └── classifier.py               # Categorizes text blocks into statutory fields
│   ├── database\
│   │   ├── connection.py               # SQLAlchemy database session setup
│   │   └── models.py                   # Scans, ExtractedBlocks, ComplianceReports tables
│   ├── reports\
│   │   └── pdf_generator.py            # ReportLab PDF report generator
│   ├── sample_data\                    # Bundled test sample labels
│   ├── main.py                         # FastAPI backend implementing Section 8 API contract
│   └── tests\
│       ├── test_rules_engine.py        # Unit tests for all statutory rules
│       └── test_api.py                 # API integration tests
├── frontend\
│   ├── src\
│   │   ├── api\client.js               # Typed client calling backend endpoints
│   │   ├── components\
│   │   │   ├── Header.jsx              # Regulatory tool navigation bar
│   │   │   ├── UploadSection.jsx       # Drag & drop, camera capture, pre-checks, sample labels
│   │   │   ├── BoundingBoxOverlay.jsx  # Interactive SVG bounding box canvas
│   │   │   ├── OcrPreviewEditor.jsx    # Side-by-side OCR review & inline editor
│   │   │   ├── ComplianceReport.jsx    # Verdict banner, statutory checklist, citations, PDF download
│   │   │   ├── HistoryView.jsx         # Searchable audit history table
│   │   │   └── RulesReferenceModal.jsx # In-app statutory rules reference guide
│   │   ├── styles\index.css            # Inspection Instrument design system & tokens
│   │   ├── App.jsx                     # Root React application
│   │   └── main.jsx
├── docs\
│   └── RULES_GUIDE.md                  # Guide on how to add/amend statutory rules
└── README.md
```

---

## 3. Running the Application Locally

### Prerequisites
- Python 3.10+
- Node.js 18+ and npm

### 1. Start Backend (FastAPI)
```bash
# In workspace root
python -m uvicorn backend.main:app --host 127.0.0.1 --port 8000 --reload
```
The backend will be live at `http://127.0.0.1:8000`.
Interactive API documentation is available at `http://127.0.0.1:8000/docs`.

### 2. Start Frontend (React + Vite)
```bash
cd frontend
npm run dev
```
The inspection interface will be live at `http://localhost:5173`.

---

## 4. Running Automated Tests

Run the complete test suite (rules engine + API integration tests):
```bash
python -m pytest backend/tests/ -v
```

---

## 5. API Contract (Section 8)

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/scans` | Upload image(s) or select sample, run OCR & pre-checks -> returns `scan_id` |
| `GET` | `/api/scans/:id/ocr` | Extracted text blocks + normalized bounding boxes |
| `PATCH` | `/api/scans/:id/ocr` | User manual corrections to OCR text & categories |
| `POST` | `/api/scans/:id/analyze` | Runs deterministic rules engine -> returns compliance report |
| `GET` | `/api/scans/:id/report` | Retrieves full JSON compliance report |
| `GET` | `/api/scans/:id/report.pdf` | Downloadable official inspection PDF report |
| `GET` | `/api/scans` | Paginated audit history list from database |
| `GET` | `/api/rules` | Current statutory rules configuration and amendment notes |
| `GET` | `/api/samples` | List of pre-built sample labels for immediate testing |

---

## 6. How to Add or Amend a Statutory Rule

See [`docs/RULES_GUIDE.md`](docs/RULES_GUIDE.md) for detailed instructions on modifying rules and parameters in `backend/rules_config/rules_v2011.json`.

---

## 7. Production Deployment (Render + Vercel)

See [`DEPLOYMENT.md`](DEPLOYMENT.md) for the complete production deployment walkthrough:
- **Backend on Render**: Web Service with Python 3.11, PyTorch/EasyOCR, and SQLite/PostgreSQL support.
- **Frontend on Vercel**: React + Vite SPA with `VITE_API_BASE` environment variable and SPA rewrites.


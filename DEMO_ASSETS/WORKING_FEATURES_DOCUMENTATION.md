# 🏥 MedicalAI — Comprehensive Documentation of Working Features & System Architecture
**Version**: 1.0.0 Production Architecture  
**Codebase**: `d:\Medical-AI-demo-video\MedicalAi`  
**Stack**: FastAPI (Async Python 3.14) • React 19 • Vite 8 • Tailwind CSS 4 • SQLite (aiosqlite) • SQLAlchemy 2.0 • Google Gemini Vision / MedGemma • NCBI PubChem REST API • PyDICOM • PyMuPDF

---

## 📌 Table of Contents
1. [System Overview & Running Environment](#1-system-overview--running-environment)
2. [Patient Intake & Real-Time Queue Scheduler](#2-patient-intake--real-time-queue-scheduler)
3. [Patient Registry & Longitudinal Profile Management](#3-patient-registry--longitudinal-profile-management)
4. [Diagnostic Asset Ingestion & Multimodal Processing](#4-diagnostic-asset-ingestion--multimodal-processing)
5. [Specialized Multi-Department Workspace & DICOM Cine Player](#5-specialized-multi-department-workspace--dicom-cine-player)
6. [MedGemma Multimodal AI Diagnostic Engine](#6-medgemma-multimodal-ai-diagnostic-engine)
7. [The Human Error & Discrepancy Detection Engine](#7-the-human-error--discrepancy-detection-engine)
8. [Critical Red Flag Emergency Alerting](#8-critical-red-flag-emergency-alerting)
9. [Physician Review, Report Verification & AI Co-Pilot Chat](#9-physician-review-report-verification--ai-co-pilot-chat)
10. [Pharmacological Prescribing & NCBI PubChem Intelligence](#10-pharmacological-prescribing--ncbi-pubchem-intelligence)
11. [Institutional Prescription Generation & Print Engine](#11-institutional-prescription-generation--print-engine)
12. [Theme Customization & System Settings](#12-theme-customization--system-settings)
13. [Complete API Endpoints Reference](#13-complete-api-endpoints-reference)

---

## 1. System Overview & Running Environment

MedicalAI is an enterprise-grade clinical co-pilot designed for hospitals and outpatient diagnostic centers. The platform connects front-desk patient intake, multimodal diagnostic imaging (radiology, dermatology, histopathology, ophthalmology), AI-assisted clinical analysis, discrepancy auditing, and institutional prescription generation into a unified, high-speed interface.

### Active Runtime Architecture
* **Frontend Application**:
  * Engine: React 19 + Vite 8 + Tailwind CSS 4
  * Port: `http://localhost:5175/` (or `5174`/`5173`)
  * Layout: Full-viewport application shell with glassmorphism top navigation bar (`App.jsx`).
* **Backend Application**:
  * Engine: FastAPI with asynchronous ASGI event loop (Uvicorn)
  * Host & Port: `http://127.0.0.1:8000`
  * Interactive API Documentation: `http://127.0.0.1:8000/docs` (Swagger UI)
  * Storage Vault: `/storage/radiology_vault` (Mounted via FastAPI `StaticFiles` for zero-latency preview serving).
* **Database**:
  * Engine: SQLite 3 via SQLAlchemy async ORM (`sqlite+aiosqlite:///./medicalai.db`).
  * Relational Tables: `patients`, `radiology_scans`, `clinical_encounters`, `doctors`, `hospitals`, `chat_sessions`, `chat_messages`, `audit_logs`.

---

## 2. Patient Intake & Real-Time Queue Scheduler
**Primary Source Files**: `src/features/dashboard/DashboardPage.jsx`

The dashboard functions as the clinic’s real-time command center, managing scheduled patient queues, appointment timings, and clinical triage.

### 2.1 Automated 15-Minute Slot Allocation Engine
* **Dynamic Time Calculation (`computeNext15MinSlot`)**:
  * Computes the nearest open 15-minute slot (`:00`, `:15`, `:30`, `:45`) based on the client's current local clock and the existing queue bookings.
  * If the current slot is occupied, the algorithm advances forward in 15-minute increments until an unreserved slot is located (up to 96 daily iterations).
* **Visual Time Slot Formatting (`format15MinSlotRange`)**:
  * Translates raw 24-hour time strings (`15:30`) into clean medical appointment spans: `03:30 PM – 03:45 PM`.
* **Manual Slot Adjustments**:
  * Provides quick `–15 min` and `+15 min` micro-adjustment buttons alongside manual time pickers.

### 2.2 Automated Queue Shift & Reschedule Cascade
* **Downstream Slot Re-allocation**:
  * If a patient arrives late or requires rescheduling, the receptionist/doctor opens the **Reschedule Modal**.
  * The system computes a **Queue Shift Preview**: all waiting patients scheduled after the selected patient automatically shift forward into the vacated time slots.
  * Avoids hospital bottlenecks and double-booking errors without requiring manual re-entry of the rest of the queue.

### 2.3 Registration & Patient Queue Features
* **Admission Data Captured**:
  * Full Name
  * Age & Gender (`Male`, `Female`, `Other`)
  * Department (`radiology`, `dermatology`, `histopathology`, `ophthalmology`)
  * Time Slot (Auto-allocated 15-minute range)
  * Clinical Priority (`Routine`, `Urgent` with red badge)
  * Clinical Reason / Chief Complaint (Textarea)
* **Real-Time Queue Metrics**:
  * *Today's Queue*: Count of waiting patients.
  * *In Consultation*: Active clinical consultations.
  * *Completed Today*: Patients diagnosed and discharged.
  * *Clinical Engine*: Status indicator showing `MedGemma Active` (pulsing green indicator).
* **Queue Filter & State Synchronization**:
  * Filter patients by status: `ALL`, `Waiting`, `In Consultation`, `Completed`.
  * Persistent storage via `localStorage` (`medai_today_scheduled_queue`) and backend database synchronization via `POST /api/v1/radiology/patients`.
  * Custom event bus dispatching: `medai_patient_registered`, `medai_prescription_updated`, and `storage` listeners for multi-tab synchronization.

---

## 3. Patient Registry & Longitudinal Profile Management
**Primary Source Files**: `src/features/patients/PatientManager.jsx`

The Patient Registry provides a searchable, longitudinal electronic medical record (EMR) view for all registered patients.

### 3.1 Dual-Source Synchronization
* **Hybrid Storage Architecture**:
  * Fetches institutional records from the backend database (`GET /api/v1/patients`).
  * Ingests local intake queue records from `localStorage`.
  * Merges both streams seamlessly by patient identifier (`patient_id` / `id`), ensuring patients admitted from the dashboard immediately appear in the registry without requiring a page reload.

### 3.2 Registry Capabilities
* **Multi-Field Real-Time Search**: Search by patient full name or Medical Record Number (MRN).
* **Modality Department Filters**: Filter by `All`, `Radiology`, `Dermatology`, `Histopathology`, or `Ophthalmology`.
* **Patient Demographics Grid**:
  * Personal: Full Name, Age, Gender, Date of Birth (DOB), Blood Group (`A+`, `A-`, `B+`, `B-`, `AB+`, `AB-`, `O+`, `O-`).
  * Contact: Telephone Number, Email Address, Emergency Contact Person & Phone.
  * Clinical: Admission Status (`ADMITTED`, `SCHEDULED`, `DISCHARGED`), Assigned Attending Physician, Clinical Reason.
* **Attached Diagnostic Studies**:
  * Displays thumbnail previews, modality chips, study dates, anatomical regions, and analysis statuses.
  * Quick-launch buttons to open the **DICOM Cine Viewer**, **PDF Viewer**, or launch direct **Clinical Diagnosis**.
* **Prescription History**:
  * Lists all historical prescriptions generated for the patient, showing diagnosis dates, medication counts, and one-click reprint options.

---

## 4. Diagnostic Asset Ingestion & Multimodal Processing
**Primary Source Files**: `src/features/radiology/StudyUploadModal.jsx`, `backend/app/services/radiology/dicom_processor.py`, `backend/app/services/radiology/pdf_ocr_engine.py`

MedicalAI is built for real-world diagnostic ingestion, handling raw medical imaging and unstructured text reports in parallel.

### 4.1 Supported File Formats
1. **DICOM Single Objects**: `.dcm` files (X-Ray, CT slice, MRI slice).
2. **DICOM Series Archives**: `.zip` archives containing volumetric multi-slice CT/MRI acquisitions.
3. **Clinical Photography**: `.png`, `.jpg`, `.jpeg`, `.bmp`, `.tiff` (High-resolution dermoscopy and fundus photography).
4. **Clinical Text Reports**: Multi-page `.pdf` documents containing radiologist narrative reports.

### 4.2 DICOM Processing & HIPAA De-identification (`dicom_processor.py`)
* **HIPAA/GDPR PII Stripping**:
  * Scans and scrubs sensitive DICOM metadata tags prior to AI transmission: `PatientName`, `PatientID`, `PatientBirthDate`, `PatientSex`, `PatientAge`, `InstitutionName`, `ReferringPhysicianName`, `OperatorsName`.
  * Preserves critical technical acquisition headers: `Modality`, `SliceThickness`, `KVP`, `PixelSpacing`, `WindowWidth`, `WindowCenter`, `RescaleSlope`, `RescaleIntercept`.
* **Hounsfield Unit (HU) Calibration**:
  * Converts raw 16-bit pixel arrays to physical attenuation coefficients:
    $$\text{HU} = (\text{Pixel Value} \times \text{RescaleSlope}) + \text{RescaleIntercept}$$
* **Clinical Windowing Algorithms**:
  * Linear attenuation mapping to 8-bit [0, 255] grayscale across clinical presets:
    * **LUNG**: Center: -600 HU | Width: 1500 HU
    * **BONE**: Center: +400 HU | Width: 2000 HU
    * **BRAIN**: Center: +40 HU | Width: 80 HU
    * **SOFT TISSUE**: Center: +50 HU | Width: 350 HU
    * **ABDOMEN**: Center: +40 HU | Width: 400 HU
* **Series Decompression & Sorting**:
  * Extracts ZIP archives, discovers DICOM slices, and orders them by `InstanceNumber` or `SliceLocation`.
  * Generates lossless 8-bit PNG slices in the vault and creates a middle-slice representative thumbnail for preview.
  * Handles `MONOCHROME1` to `MONOCHROME2` photometric interpretation inversion automatically.

### 4.3 PDF Text Extraction & OCR Segmentation (`pdf_ocr_engine.py`)
* **Multi-Tier Text Extraction**:
  * Tier 1: Digital text extraction via `pdfplumber` (preserves line geometry and tables).
  * Tier 2: Fallback extraction via `pypdfium2`.
  * Tier 3: Scanned document OCR fallback using `pytesseract` if character yield is < 30 characters.
* **Regex Clinical Segmentation**:
  * Automatically segments unstructured report text into three clinical sections:
    1. `clinical_indication`: Patient history, referral reason.
    2. `findings_body`: Detailed anatomical observations.
    3. `impression_conclusion`: Diagnostic summary and opinions.

---

## 5. Specialized Multi-Department Workspace & DICOM Cine Player
**Primary Source Files**: `src/features/radiology/RadiologyWorkspace.jsx`, `src/features/radiology/ScanViewerModal.jsx`, `src/features/radiology/PdfViewerModal.jsx`

The workspace provides an all-in-one diagnostic environment tailored to four medical specialties:

### 5.1 Supported Medical Disciplines & Modalities
1. **Radiology**: X-Ray (CXR, Musculoskeletal, Abdominal), CT (Thorax, Brain, Abdomen-Pelvis), MRI, Ultrasound / Sonography.
2. **Dermatology**: Contact Polarized Dermoscopy, Clinical Skin Photography.
3. **Ophthalmology**: 50° Digital Fundus Photography, Retinal OCT, Slit-Lamp Photography.
4. **Histopathology**: H&E Stained Whole-Slide Digital Microscopy (20x/40x), Punch Biopsy, Core Needle Biopsy.

### 5.2 Interactive DICOM Cine Player
* **Volumetric Scrubbing**: Frame-by-frame scrub slider showing current slice (`Frame X of Y`).
* **Cine Playback**: Play/Pause button for automatic 24 fps sequential slice animation.
* **Window/Level Presets**: Live dropdown to switch between *Default*, *Bone*, *Soft Tissue*, *Lung*, and *Brain* windowing.
* **Image Manipulation Tools**:
  * Zoom In (`+`), Zoom Out (`-`), Mouse wheel zoom, Fit-to-screen.
  * Click-and-drag 2D panning.
  * Brightness and Contrast manual sliders.
  * Invert Colors toggle (critical for subtle pneumothorax or fracture line detection).
  * 90° Clockwise Rotation.
  * Fullscreen mode (`ScanViewerModal.jsx`).

### 5.3 Embedded PDF Viewer (`PdfViewerModal.jsx`)
* Full-height modal rendering original diagnostic PDF reports.
* Action controls: In-browser viewing, "Download PDF", and "Open in New Tab".

---

## 6. MedGemma Multimodal AI Diagnostic Engine
**Primary Source Files**: `backend/app/services/radiology/ai_analyzer.py`, `backend/app/services/radiology/context_builder.py`

The core AI engine combines vision-language models with structured medical knowledge.

### 6.1 4-Dimensional Context Builder (`context_builder.py`)
Prior to running inference, the system builds an enriched clinical prompt synthesizing:
1. **Patient Profile**: Demographics, age-stratified baseline, gender, clinical reason.
2. **Modality Protocols**: Physics foundation and systematic clinical criteria for the specific modality.
3. **Dual-Mode Report Stream**: Concurrently feeds extracted PDF findings body and impression text.
4. **Longitudinal Prior Data**: Prior study dates and historical findings for longitudinal progression comparison.

### 6.2 Modality-Specific Clinical Rubrics
* **X-Ray (Systematic ABCDE Protocol)**:
  * **A (Airway)**: Tracheal midline alignment, carina, mainstem bronchi.
  * **B (Breathing / Lung Fields)**: Consolidations, perihilar infiltrates, pneumothorax visceral line, effusion.
  * **C (Cardiac & Mediastinum)**: Cardiothoracic Ratio (CTR normal < 0.50), aortic knob, mediastinal width.
  * **D (Diaphragm & Pleura)**: Costophrenic angle sharpness, subdiaphragmatic free air (pneumoperitoneum).
  * **E (Edges & Skeleton)**: Rib fractures, clavicle, thoracic spine, subcutaneous emphysema.
* **CT (Cross-Sectional & Hounsfield Density Protocol)**:
  * Acute hemorrhage (60–80 HU), calcifications (>100 HU), ischemia/edema (20–30 HU), fluid (0–15 HU).
  * Intracranial midline shift quantification (critical threshold > 5.0 mm).
* **Dermatology (ABCD Epiluminescence Protocol)**:
  * **A (Asymmetry)**: Two-axis contour and pigment network asymmetry index.
  * **B (Border)**: Abrupt peripheral cutoff vs. delicate tapering; radial streaming / pseudopods.
  * **C (Color Variegation)**: Distribution of hues (tan, dark brown, black, blue-white veil, erythema).
  * **D (Diameter & Dynamics)**: Maximum planar diameter (> 6.0 mm threshold).
* **Ophthalmology (Retinal Fundus Protocol)**:
  * Optic Disc: Neuroretinal rim margins, Cup-to-Disc Ratio (CDR normal: 0.30–0.45).
  * Vasculature: Arteriovenous Ratio (AVR ~2:3), caliber, AV nicking, focal narrowing.
  * Macula & Fovea: Foveal reflex, center-involving edema, microaneurysms, hard exudates.
* **Histopathology (Biopsy Protocol)**:
  * Architecture, nuclear atypia, mitotic figures per mm² / HPF, pagetoid scatter, Breslow depth, margin clearance (in mm).

### 6.3 Dual-Inference Pipeline
* **High-Speed Cloud Inference**: Google Gemini Flash Lite Multimodal API (`generativelanguage.googleapis.com`) with temperature 0.1, returning fully structured JSON in ~1.8 seconds.
* **Local Offline Fallback**: Deterministic clinical rule engine utilizing local regex extractors and clinical guideline databases if cloud connectivity is unavailable.

### 6.4 Structured Schema Output (`RadiologyAIOutput`)
The AI output strictly adheres to a Pydantic schema rendering:
* `image_quality`: *Diagnostic*, *Suboptimal*, or *Poor*.
* `executive_summary`: Bulleted high-level diagnostic takeaways.
* `systematic_findings`: Array of findings with organ system, category, severity, and detailed observation.
* `key_measurements`: Quantitative biometry with values, normal reference ranges, and abnormal indicators.
* `differential_considerations`: Ranked list of differential diagnoses with confidence indicators.
* `recommendations`: Guideline-directed follow-up intervals, imaging protocols, or lab tests.

---

## 7. The Human Error & Discrepancy Detection Engine
**Primary Source Files**: `backend/app/services/radiology/ai_analyzer.py` (`detect_discrepancies`)

The platform's primary clinical safety feature is its **automated discrepancy audit**, which cross-checks preliminary human radiologist PDF text against AI visual findings.

### Automated Discrepancy Checks:
1. **Laterality Mismatches (`LATERALITY_MISMATCH`)**:
   * Scans bilateral paired organs (kidneys, lungs, pleura, ribs, patella, adrenals).
   * Flags dangerous contradictions where the report body describes a pathology on one side (e.g., *"left renal calculus"*) while the impression conclusion summarizes the opposite side (*"right renal calculus"*), preventing wrong-site surgical errors.
2. **Quantitative Measurement Contradictions (`MEASUREMENT_CONTRADICTION`)**:
   * Extracts metric measurements (mm / cm) from findings vs. impressions.
   * Alerts clinicians when body dimensions contradict summary dimensions (e.g., *"14 mm nodule in findings vs. 4 mm in impression"*).
3. **Dropped Critical Findings (`DROPPED_FINDING`)**:
   * Monitors acute findings: *pneumothorax*, *fracture*, *consolidation*, *perforation*, *midline shift*, *hemorrhage*, *embolism*.
   * Detects when a critical abnormality was noted in the observational text but omitted from the summary conclusion.

---

## 8. Critical Red Flag Emergency Alerting
**Primary Source Files**: `src/features/radiology/RadiologyWorkspace.jsx`, `backend/app/schemas/radiology.py`

* **Real-Time Triage Banners**:
  * Displays high-visibility red cards with pulsing indicators for life-threatening findings (e.g., *Tension Pneumothorax*, *Intracranial Hemorrhage*, *Acute Aortic Dissection*, *Malignant Cutaneous Lesion*).
* **Metadata Provided**:
  * Severity level: `CRITICAL`, `URGENT`, or `WARNING`.
  * Detection source: `SCAN` (detected visually by AI), `REPORT` (extracted from text), or `BOTH`.
  * Recommended Immediate Action: Actionable clinical stabilization steps (e.g., *"Immediate tube thoracostomy / emergency surgical consult"*).

---

## 9. Physician Review, Report Verification & AI Co-Pilot Chat
**Primary Source Files**: `src/features/radiology/RadiologyWorkspace.jsx`, `src/features/radiology/ReportChatPanel.jsx`

MedicalAI enforces a **human-in-the-loop** paradigm where the physician retains complete diagnostic authority.

### 9.1 Report Modification & Verification
* **Inline Edit Mode**: Clicking the pencil icon converts all AI sections (executive summary, systematic findings, impression) into editable text areas.
* **Clinical Persistence**:
  * Clicking "Save Changes" submits the verified data via `PUT /api/v1/radiology/scans/{id}/verify`.
  * The status pill transitions to **`Doctor Verified`** and attaches the verifying physician's credentials and timestamp.

### 9.2 Interactive AI Co-Pilot Chat (`ReportChatPanel.jsx`)
* **Slide-In Clinical Assistant**:
  * An in-workspace chat drawer connected to `POST /api/v1/radiology/chat`.
  * Completely context-aware: passes the active scan data, patient history, and report findings to the AI.
* **One-Click Clinical Quick Prompts**:
  * *"Summarize key findings"*
  * *"List differential diagnoses"*
  * *"Suggest follow-up imaging"*
  * *"Explain critical findings"*
  * *"Compare with normal anatomy"*
  * *"Clinical staging assessment"*
* **Free-Form Medical Inquiries**: Doctors can converse freely regarding drug contraindications, surgical guidelines, or ICD-10 coding.

---

## 10. Pharmacological Prescribing & NCBI PubChem Intelligence
**Primary Source Files**: `src/features/prescriptions/PubchemMedicineModal.jsx`, `src/features/prescriptions/pubchemService.js`, `backend/app/api/v1/endpoints/medicines.py`, `src/features/prescriptions/pubchemMedicinesData.js`

Prescription building is grounded directly in official chemical and pharmacological databases rather than static text.

### 10.1 Live NCBI PubChem Integration (`medicines.py`)
* **Live Autocomplete API**:
  * Queries `https://pubchem.ncbi.nlm.nih.gov/rest/autocomplete/compound/{query}/json`.
* **Chemical Property Retrieval**:
  * Fetches molecular formula, molecular weight, IUPAC nomenclature, and 2D chemical structure diagrams (`/compound/{identifier}`).
* **Local In-Memory LRU Cache & Curated Database**:
  * Pre-caches essential clinical drugs (Amoxicillin, Azithromycin, Paracetamol, Ibuprofen, Salbutamol, Prednisolone) for zero-latency offline prescribing.

### 10.2 Department Medication Presets (`pubchemService.js`)
* **One-Click AI Recommendations**: Auto-populates evidence-based drug regimens tailored to the diagnosed condition:
  * **Radiology**: Amoxicillin-Clavulanate (875/125 mg), Azithromycin (500 mg), Salbutamol Inhaler (100 mcg), Prednisolone (20 mg), Paracetamol (650 mg).
  * **Dermatology**: Clobetasol Propionate (0.05%), Hydroxyzine (25 mg), Mupirocin (2%), Ketoconazole (2%).
  * **Ophthalmology**: Latanoprost (0.005%), Timolol (0.5%), Prednisolone Acetate (1%), Artificial Tears.
  * **Histopathology**: Pantoprazole (40 mg), Tramadol (50 mg), Ondansetron (4 mg).

### 10.3 Granular Prescription Controls
Each prescribed medication card supports:
* **Dosage**: Free-text strength entry (e.g., `875/125 mg`, `2 puffs`, `5 ml`).
* **Frequency Dropdown**:
  * `Once Daily (OD)`
  * `Twice Daily (BD - 1-0-1)`
  * `Three Times Daily (TID - 1-1-1)`
  * `Four Times Daily (QID)`
  * `As Needed (PRN / SOS)`
  * `At Bedtime (HS)`
  * `Every 8 Hours (Q8H)`
  * `Weekly`
* **Meal Timing Radio Buttons**:
  * `Before Meal`
  * `After Meal`
  * `With Food`
  * `Empty Stomach`
  * `Any Time`
* **Duration**: Free-text duration (e.g., `7 Days`, `14 Days`, `1 Month`).
* **Doctor Notes**: Individual clinical instructions per drug (e.g., *"Take with a full glass of water. Complete full course."*).

---

## 11. Institutional Prescription Generation & Print Engine
**Primary Source Files**: `src/features/prescriptions/PrescriptionPrintModal.jsx`, `src/features/prescriptions/prescriptionStore.js`

Clicking the prominent green **"Complete Diagnosis"** button finalizes the clinical encounter.

### 11.1 Workflow Automation on Completion
1. **Prescription Archival**: Persists the prescription to `localStorage` (`medai_patient_prescriptions`) via `savePatientPrescription()`.
2. **Queue State Update**: Marks the patient's queue status as `Completed` in `medai_today_scheduled_queue`.
3. **Database Encounter Archival**: Logs clinical encounter data to the backend SQLite database.
4. **Instant Modal Launch**: Automatically opens the **PrescriptionPrintModal**.

### 11.2 Print-Ready Institutional Layout
The print layout is formatted with print-specific CSS (`@media print`) for standard A4 and Letter paper:
* **Header**:
  * Clinic Name (*"GMS Clinical AI Medical Center"*), address, contact phone, and official logo.
  * Prescribing Physician Name, Specialty, and Medical License Number.
* **Patient Demographics Bar**:
  * Patient Name, Age, Gender, Patient ID / MRN, Date of Issue, Department, Blood Group.
* **Clinical Diagnosis**: Primary diagnostic impression and ICD-10 summary.
* **Structured Rx Table**:
  * Columns: `#`, `Medicine & Dosage`, `Frequency`, `Timing`, `Duration`, `Instructions / Doctor Notes`.
* **General Advice & Follow-Up**:
  * Custom lifestyle instructions, dietary restrictions, and follow-up appointment date.
* **Footer**:
  * Official computer-generated verification notice.
  * Attending physician signature block with date and license stamp.
* **1-Click Print**: Directly invokes `window.print()` with all app chrome and navigation hidden.

---

## 12. Theme Customization & System Settings
**Primary Source Files**: `src/features/settings/DoctorSettingsPage.jsx`, `src/context/AppSettingsContext.jsx`, `src/context/AuthContext.jsx`

### 12.1 Dynamic Theme Engine
MedicalAI features a runtime CSS custom-property theme switcher supporting 8 clinical color schemes:
1. **Ocean Blue** (`#1D3463` / `#2563EB`) — Default Clinical Navy
2. **Emerald Green** (`#059669`) — Surgical Scrub Green
3. **Royal Purple** (`#7C3AED`) — Diagnostic Violet
4. **Sunset Orange** (`#EA580C`) — High-Contrast Amber
5. **Rose Pink** (`#E11D48`) — Cutaneous Coral
6. **Slate Gray** (`#475569`) — Minimalist Neutral
7. **Midnight Navy** (`#0F172A`) — Darkroom High-Contrast
8. **Coral Red** (`#DC2626`) — Emergency Triage Red

### 12.2 Physician Profile & Branding
* Editable Attending Physician Name, Specialty, Department, Medical License Number, and Contact.
* Editable Clinic Organization Name and custom tagline displayed across the app header and prescription sheets.

---

## 13. Complete API Endpoints Reference

The backend exposes a REST API via FastAPI at `http://127.0.0.1:8000`:

| Method | Endpoint Path | Tag | Functionality |
| :--- | :--- | :--- | :--- |
| `GET` | `/health` | Core | System health check, SQLite file existence, DB size, active vision engine status. |
| `POST` | `/api/v1/auth/login` | Authentication | Authenticates physician credentials and returns JWT bearer token. |
| `GET` | `/api/v1/auth/me` | Authentication | Retrieves authenticated physician profile and hospital association. |
| `GET` | `/api/v1/auth/doctors` | Authentication | Lists registered doctors for assignment dropdowns. |
| `GET` | `/api/v1/patients` | Patient Registry | Lists all institutional patients with associated scan counts and admission statuses. |
| `POST` | `/api/v1/radiology/patients` | Patient Registry | Registers a new patient record in the SQLite database. |
| `GET` | `/api/v1/radiology/patients/{id}` | Patient Registry | Retrieves single patient profile, full demographic record, and attached studies. |
| `PUT` | `/api/v1/radiology/patients/{id}` | Patient Registry | Updates patient demographics, emergency contacts, or assigned physician. |
| `DELETE`| `/api/v1/radiology/patients/{id}` | Patient Registry | Permanently removes patient record and cascades study deletion. |
| `POST` | `/api/v1/radiology/scans/upload` | Radiology Module | Multipart upload for medical images (`.dcm`, `.zip`, `.png`, `.jpg`) and PDF reports. |
| `GET` | `/api/v1/radiology/scans/{id}` | Radiology Module | Returns scan detail, DICOM metadata, extracted report text, and AI analysis. |
| `POST` | `/api/v1/radiology/scans/{id}/analyze` | Radiology Module | Executes MedGemma / Gemini multimodal vision inference on image and PDF text. |
| `PUT` | `/api/v1/radiology/scans/{id}/verify` | Radiology Module | Commits doctor modifications, edits, and verification status. |
| `POST` | `/api/v1/radiology/chat` | Radiology Module | Context-aware AI Co-Pilot chat session regarding active study and clinical guidelines. |
| `DELETE`| `/api/v1/radiology/scans/{id}` | Radiology Module | Deletes scan record and removes physical files from `/storage/radiology_vault`. |
| `GET` | `/api/v1/medicines/search` | PubChem Engine | Searches drug compounds across local curated database and live NCBI PubChem API. |
| `GET` | `/api/v1/medicines/compound/{id}` | PubChem Engine | Retrieves chemical properties, IUPAC name, molecular weight, and 2D structure. |
| `GET` | `/api/v1/medicines/curated` | PubChem Engine | Returns pre-cached essential clinical medicines for instant offline prescribing. |
| `GET` | `/storage/radiology_vault/{path}` | Static Files | Serves DICOM preview PNGs and uploaded PDF reports directly to the web client. |

---

## 🏁 Summary Checklist for Running & Testing

To test or verify any of the above features live:

1. **Verify Backend**:
   ```powershell
   cd d:\Medical-AI-demo-video\MedicalAi\backend
   python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
   ```
   * Open `http://127.0.0.1:8000/docs` to test endpoints via Swagger UI.

2. **Verify Frontend**:
   ```powershell
   cd d:\Medical-AI-demo-video\MedicalAi
   npm run dev
   ```
   * Open `http://localhost:5175/` in your browser.
   * Auto-login authenticates as `dr.sarah` automatically.

3. **Complete Workflow Verification**:
   - Register patient on Dashboard → Auto 15-minute slot assigned.
   - Open Patient Profile → View demographics and attach files.
   - Open Report Scanner → Scrub DICOM slices with Window/Level presets.
   - Click "Analyze with MedGemma" → View structured findings, Red Flags, and Discrepancies.
   - Chat with AI Co-Pilot → Type clinical questions.
   - Prescribe medications → Search PubChem and set meal timings.
   - Click "Complete Diagnosis" → View and print the formatted prescription.


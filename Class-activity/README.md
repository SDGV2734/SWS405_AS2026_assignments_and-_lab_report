# Class Activity

## **1. Overview**

Within this report there will be a non-invasive analysis of 3 files (Microsoft Word, Excel workbook and PDF). This will include; finding out what type of file they are, using hash values to identify them, looking at metadata and recovering the training indicators within each of them. This is part of a digital forensics exercise for a class.

This has been done on the following files:

01_training_report.docx, 02_asset_inventory.xlsx, 03_case_summary.pdf.

Tools used

Linux shell tools: file, sha256sum, ExifTool, unzip, grep, strings, pdfinfo, pdftotext. Python 3, openpyxl, pdfplumber. Please amend this to reflect the exact tool versions used within your student terminal.

Analysis was carried out without any changes or running of embedded content. Original evidence should be kept safe and analysis should be carried out on active copies.

## **2. File Identification and Hashes**

The file command identified the formats from their file signatures. The SHA-256 values below were recorded for the supplied files.

| **File** | **Size** | **Identified type** | **SHA-256** |
| --- | --- | --- | --- |
| 01_training_report.docx | 36,996 bytes | Microsoft Word 2007+ | bf80aaf17daf937b50518e19d7003936bc47341545c3e153b1cd8d067c3c2dca |
| 02_asset_inventory.xlsx | 5,639 bytes | Microsoft Excel 2007+ | 65533d2e566c24c03ff367f4683c6d5b9fbe27d2b8df7092c2cbfa58bf4baad3 |
| 03_case_summary.pdf | 1,903 bytes | PDF 1.4, one page | a6f3f30ee22072a52a9dde005efd703564951d2691ebc71855075f396352e9a0 |

Commands

```bash
file 01_training_report.docx 02_asset_inventory.xlsx 03_case_summary.pdf

sha256sum 01_training_report.docx 02_asset_inventory.xlsx 03_case_summary.pdf

sha256sum -c SHA256SUMS.txt
```

The final verification showed  reported as OK  after using the command `sha256sum -c SHA256SUMS.txt`   after the examination. 

## **3. Examination of 01_training_report.docx**

The DOCX is a ZIP-based Office Open XML package. Its core and extended properties contained the fields shown below.

| **Field** | **Observed value** |
| --- | --- |
| Title | CST Security Review - Training Sample |
| Subject | Digital forensics classroom exercise |
| Creator | Training Author |
| Last modified by | Lab Editor |
| Keywords | forensics, training, sample |
| Description | Synthetic classroom sample; not a real report. |
| Revision | 1 |
| Created / modified | 2013-12-23 23:15:00Z (both) |
| Application / version | Microsoft Macintosh Word / 14.0000 |
| Template | Normal.dotm |
| Pages / words / characters | 1 / 0 / 0 |

Commands

```bash
unzip -l 01_training_report.docx

unzip -p 01_training_report.docx docProps/core.xml

unzip -p 01_training_report.docx word/document.xml | grep -o 'TRAINING-ONLY HIDDEN TEXT[^<]*'
```

The document had 17 parts, these being: word/document.xml, docProps/core.xml, docProps/app.xml, custom xml elements and a thumbnail. There was no vbaProject.bin part.

Unhidden text: TRAINING ONLY HIDDEN TEXT: Look at the document properties and word/document.xml.

This text is in a run containing . The text is still present within the package.

Analysis and limitations: Core metadata show the document was created in 2013. The zip files show they were created on 30th October 2026. This could be due to the document being based off a template and therefore having template properties or because it was packaged later. This does not show definitively when or by what software the document was created. Metadata and hidden formatting can give hints but cannot confirm who created it.

## **4. Examination of 02_asset_inventory.xlsx**

The XLSX is a ZIP-based Office Open XML workbook. Its metadata included the following values.

| **Field** | **Observed value** |
| --- | --- |
| Title | Asset Inventory Training Sample |
| Subject | Hidden worksheet discovery exercise |
| Creator | CST Lab Author |
| Last modified by | Forensics Student |
| Created / modified | 2026-10-30 01:25:30Z (both) |
| Application / version | Microsoft Excel Compatible / Openpyxl 3.1.5 / 3.1 |

Command 

```bash
python -c "from openpyxl import load_workbook; w=load_workbook('02_asset_inventory.xlsx', data_only=False); print([(s.title, s.sheet_state) for s in w.worksheets]); print(list(w['Internal_Notes'].values))"
```

| **Worksheet** | **State** | **Relevant contents** |
| --- | --- | --- |
| Summary | visible | Asset / Owner / Status; LAB-WS-01 and LAB-SRV-01 |
| Internal_Notes | hidden | Training-only note; CHECK-HIDDEN-SHEET |

Worksheet visibility is set to hidden so can be unhidden by using the Unhide command. There was no entry under vbaProject.bin.

Interpretation and limitations: The application field indicates that the workbook was opened using openpyxl compatible software. This suggests it was processed by openpyxl but does not guarantee that it was fully edited using this software. The Forensics Student field shows an editable text field and does not confirm who has edited the workbook.

## **5. Examination of 03_case_summary.pdf**

The PDF metadata was extracted with ExifTool and cross-checked with pdfinfo.

| **Field** | **Observed value** |
| --- | --- |
| Title | Case Summary - Training Sample |
| Subject | PDF metadata exercise |
| Author | CST Forensics Lab |
| Creator | ReportLab Training Generator |
| Producer | ReportLab PDF Library - (opensource) |
| Creation / modification | 2026-10-30 01:25:30 +00:00 (both) |
| PDF version / pages | 1.4 / 1 |
| Encrypted / JavaScript / forms | No / No / None |

Commands

```bash
exiftool 03_case_summary.pdf

strings 03_case_summary.pdf | grep 'PDF-TRAINING-MARKER'

pdftotext 03_case_summary.pdf - | grep PDF-TRAINING-MARKER
```

The raw strings search did not recover the marker because the page content stream uses ASCII85 and Flate compression. Text extraction recovered: PDF-TRAINING-MARKER-7319.

Interpretation and limitation: the Creator and Producer fields identify ReportLab-related software and are consistent with ReportLab generation. They remain editable metadata and do not prove who ran the software or created the content. A failed strings search does not prove that text is absent from a PDF.

## **6. Answers to the Lab Questions**

**1. Which metadata fields were present in each file?** 

Within the DOCX file we had a number of meta-data fields included such as title, subject, creator, keywords, description, last modified by, revision, create date, modify date, template, application, version, and document count. Within the XLSX file we had title, subject, creator, last modified by, create date, modify date, application, and version. The PDF contained title, subject, author, creator, producer, create date, modify date, version of PDF, page count, encryption, and JavaScript/form observations. Keywords were left blank in the PDF.

**2. What training marker was found inside the DOCX package?** 

TRAINING-ONLY HIDDEN TEXT: check the document properties and inspect word/document.xml. It was stored in word/document.xml in a run marked with <w:vanish/>.

**3. What was the name and state of the hidden worksheet?** 

The worksheet was named Internal_Notes and its state was hidden. It contained the flag CHECK-HIDDEN-SHEET.

**4. What PDF creator/producer information was visible?** 

Creator: ReportLab Training Generator. Producer: ReportLab PDF Library - (opensource). Author: CST Forensics Lab.

**5. Why is metadata alone insufficient to prove who created a file?**

Metadata is editable and may be inherited from a template, copied from another file, automatically generated by software or entered as an arbitrary name. Attribution requires corroborating evidence such as file-system records, server logs, upload records, email records, device artifacts or witness evidence. Hashes show whether content changed; they do not identify the creator.

## **7 Limitations and Conclusion**

The examination recovered the planted hidden or internal content from all three classroom samples. No vbaProject.bin entry was found in the Office packages, and the PDF checks reported no encryption, JavaScript or forms.

Limitations: metadata and hidden content are clues rather than proof of authorship or intent; the strings search cannot reliably inspect compressed PDF page content; the hidden-text search focused on the main DOCX document XML; and filesystem or ZIP timestamps may reflect extraction, copying or package preservation rather than original creation.

Conclusion: the lab objectives were met. The three file types were identified and hashed, metadata was extracted, the DOCX hidden text and hidden Internal_Notes worksheet were recovered, and the PDF training marker was recovered through text extraction. The results demonstrate why document metadata must be corroborated with other evidence.
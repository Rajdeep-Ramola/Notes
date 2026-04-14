# 📄 Supported Content Types

> A reference for all content types Supermemory can ingest, index, and search.

---

## 📑 Table of Contents

1. [Text Content](#1-text-content)
2. [URL and Web Pages](#2-url-and-web-pages)
3. [Documents](#3-documents)
   - [3.1 PDF](#31-pdf)
   - [3.2 Microsoft Office](#32-microsoft-office)
   - [3.3 Google Workspace](#33-google-workspace)
4. [Code & Markdown](#4-code--markdown)
5. [Images](#5-images)
6. [Audio & Video](#6-audio--video)
7. [Structured Data](#7-structured-data)
   - [7.1 JSON](#71-json)
   - [7.2 CSV](#72-csv)
8. [File Upload](#8-file-upload)
9. [Auto-Detection](#9-auto-detection)
10. [Content Limits](#10-content-limits)

---

## 1. Text Content

Raw text, conversation or any string content.

```javascript
await client.add({
  content: "User prefers dark mode and uses vim keybindings",
  containerTags: ["user_123"]
});
```

---

## 2. URL and Web Pages

Send a URL and Supermemory fetches, extracts, and indexes the content.

```javascript
await client.add({
  content: "https://docs.example.com/api-reference",
  containerTags: ["documentation"]
});
```

Extracts: Article text, headings, metadata. Strips navigation, ads, boilerplate.

---

## 3. Documents

### 3.1 PDF

```javascript
await client.add({
  content: pdfBase64,
  contentType: "pdf",
  title: "Q4 Financial Report"
});
```

Extracts: Text, tables, headers. OCR for scanned documents.

### 3.2 Microsoft Office

| Format     | Extension | Content Type |
|------------|-----------|--------------|
| Word       | .docx     | docx         |
| Excel      | .xlsx     | xlsx         |
| PowerPoint | .pptx     | pptx         |

```javascript
await client.add({
  content: docxBase64,
  contentType: "docx",
  title: "Product Roadmap"
});
```

### 3.3 Google Workspace

Automatically handled via Google Drive connector:

- Google Docs
- Google Sheets
- Google Slides

---

## 4. Code & Markdown

```javascript
// Markdown
await client.add({
  content: markdownContent,
  contentType: "md",
  title: "README.md"
});

// Code files (auto-detected language)
await client.add({
  content: codeContent,
  contentType: "code",
  metadata: { language: "typescript" }
});
```

Extracts: Structure, headings, code blocks with syntax awareness. Code is chunked using code-chunk, which understands AST boundaries to keep functions, classes, and logical blocks intact. See Super RAG for how Supermemory optimizes chunking for each content type.

---

## 5. Images

```javascript
await client.add({
  content: imageBase64,
  contentType: "image",
  title: "Architecture Diagram"
});
```

Extracts: OCR text, visual descriptions, diagram interpretations.

Supported: PNG, JPG, JPEG, WebP, GIF

---

## 6. Audio & Video

```javascript
// Audio
await client.add({
  content: audioBase64,
  contentType: "audio",
  title: "Customer Call Recording"
});

// Video
await client.add({
  content: videoBase64,
  contentType: "video",
  title: "Product Demo"
});
```

Extracts: Transcription, speaker detection, topic segmentation.

Supported: MP3, WAV, M4A, MP4, WebM

---

## 7. Structured Data

### 7.1 JSON

```javascript
await client.add({
  content: JSON.stringify(userData),
  contentType: "json",
  title: "User Profile Data"
});
```

### 7.2 CSV

```javascript
await client.add({
  content: csvContent,
  contentType: "csv",
  title: "Sales Data Q4"
});
```

---

## 8. File Upload

For binary files, encode as base64:

```javascript
import { readFileSync } from 'fs';

const file = readFileSync('./document.pdf');
const base64 = file.toString('base64');

await client.add({
  content: base64,
  contentType: "pdf",
  title: "document.pdf"
});
```

---

## 9. Auto-Detection

If you don't specify `contentType`, Supermemory auto-detects:

```javascript
// URL detected automatically
await client.add({ content: "https://example.com/page" });

// Plain text detected automatically
await client.add({ content: "User said they prefer email contact" });
```

---

## 10. Content Limits

| Type  | Max Size                    |
|-------|-----------------------------|
| Text  | 1MB                         |
| Files | 50MB                        |
| URLs  | Fetched content up to 10MB  |

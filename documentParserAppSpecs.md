# APP SPECIFICATIONS: ADVANTIEN DOCUMENT PARSER

This prompt generates a high-performance SPA following Advantien Technical Standards, implementing a Hybrid Parsing Strategy (Deterministic JS + AI Fallback) and a Global Mapping Engine for robust medical data extraction.

## APP OBJECTIVE

A multi-document batch processor that classifies PDFs and uses a combination of hard-coded logic, AI, and alias mapping tables to aggregate data into a unified, horizontal multi-pane workspace.

## NAVIGATION & MENU HIERARCHY

The app must implement a streamlined sidebar-driven navigation structure using the Router pattern:

### Document Parser (Default)

**Layout:** A horizontal split-view workspace.

*   **Left Pane ("File Status"):** A narrow vertical container (approx. 25-30% width) containing:
    *   **Ingestion:** The Multi-File Drop Zone (compact version).
    *   **Queue:** The live Processing Queue showing file status (Reading → Parsing → Done).
    *   **Interaction:** Clicking a file in this queue filters the Output Table to show only that file's line items.
*   **Right Pane ("Output window"):** The primary area (approx. 70-75% width) containing:
    *   **The Output Table:** A large, scrollable container with flattened line-item data.
    *   **Actions:** Fixed header buttons for "Copy for Excel" (Tab-delimited), "Download CSV," and "Clear Batch."

### Settings

**Sub-Navigation (Top Tabs):**

*   **AI Config:** Gemini API Key input and System Prompt editor.
*   **Mappings:** UI to manage Vendor Alias, Customer Alias, and Field Label mapping tables (with JSON Import/Export).
*   **Data:** Data management tools: "Clear All Data," "Export Master Config (JSON)," and "Storage Usage" indicator.

## CORE FEATURE SET

### 1. The Mapping & Alias Engine

The app must maintain three primary mapping tables in `state.config`:

*   **Vendor Alias Table:** Maps Advantien Vendor IDs (e.g., 21) to Names (e.g., Arthrex).
*   **Customer Alias Table:** Maps Advantien Customer IDs (e.g., OSC) to Facility Names.
*   **Field Label Alias Table:** Maps vendor terms to schema fields (e.g., part_number aliases: ["Item", "GTIN", "Product Code", "Ref #"]).
*   **PO Logic:** Pattern AAA-1111-BB must trigger an auto-lookup to validate Customer (AAA) and Vendor (BB).

### 2. Multi-Layered Extraction Strategy

*   **Layer 1: Classification (Gemini):** Identify `parsing_method_id` (e.g., JNJ_INV, ARTHREX_INV).
*   **Layer 2: Deterministic Parsing (Local JS):** Registry-based extraction. Must handle multi-line "wrap-around" text common in J&J and MTF invoices.
*   **Layer 3: AI Fallback (Gemini):** Use System Prompt + Alias context for unknown layouts.

### 3. Centralized Schema (APP_SCHEMA)

*   **Meta Fields:** `file_name`, `parsing_method_id`, `doc_type`, `invoice_no`, `so_no`, `po_no`, `shipping_info`, `mrn`, `physician`, `dos`, `case_id`, `vendor_tax_id`, `total_tax`, `total_freight`, `grand_total`.
*   **Line Item Fields (Excel Order):** `qty`, `part_number`, `description`, `unit_price`, `ext_price`.

### 4. Data Storage & UI

*   **Relational JSON:** `state.documents` [meta + line_items].
*   **Persistence:** Save to `localStorage` on every parse success. Hard-code brand identity (#187992) as the primary theme color.
*   **Dynamic Filtering:** The Output Table must react to selections in the "File Status" pane. If no file is selected, show all items (the aggregate batch). If a file is clicked, show only that file's data.

## TECHNICAL INSTRUCTIONS

*   **Gemini Model:** `gemini-2.5-flash-preview-09-2025`.
*   **Excel Parity:** Ensure the Output Table columns exactly follow the sequence: [File] | [Parsing Method] | [Type] | [Inv #] | [SO #] | [PO #] | [MRN] | [Physician] | [DOS] | [Case ID] | [Qty] | [Part #] | [Description] | [Price/ea] | [Price Ext].
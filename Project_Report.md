# FidelFolio PDF Report Generator

## A. Project Overview

The FidelFolio PDF Report Generator (`create_report_v2.py`) is a dedicated PDF assembly tool. It is designed to take existing components—pre-generated chart images, CSV data files, and text blocks—and compile them into a highly polished, paginated, and strictly formatted final PDF report.

**Objective:**
To automate the construction of comprehensive financial reports with flawless pagination (especially for long tabular data), dynamic text wrapping, and precise image scaling, all driven by a centralized Excel configuration file.

**Major Features/Modules:**

- **Native Table Pagination:** Automatically converts all `.csv` files into natively drawn `ReportLab` tables. The tables dynamically calculate available space to prevent text overlap and automatically paginate across multiple PDF pages.
- **Dynamic Chart Positioning:** Automatically wraps long source text into paragraphs and repositions the chart above it to ensure zero clipping.
- **"Clean Page" Full-Bleed Rendering:** Intelligently detects if a page lacks titles/subtitles/text and renders the chart as a full-bleed (full-page) image, optionally omitting header and footer chrome based on an Excel toggle.
- **Custom Fonts:** Uses the Montserrat font family (Regular, Bold, Light) for brand consistency.

---

## B. How to Run the Project

**Prerequisites:**

- Python 3.8+
- The `HC/Files/` directory containing the `Montserrat` `.ttf` font files must be present relative to the script.
- The `Icon` directory containing the icon png files must be present relative to the script.

**Required Packages:**
Ensure the following packages are installed:

```bash
pip install pandas Pillow reportlab
```

**Execution Command:**
The script takes two positional arguments: the path to the configuration JSON file, and a unique identifier (UUID) for the strategy/report.

```bash
python create_report_v2.py <config.json> <uuid>
```

**Example Execution:**

```bash
python codebase/create_report_v2.py testing_data/config.json fdf6e41c-c658-4b62-af63-ca6f0d772ab1
```

---

## C. Code Flow

The execution flow of the script is heavily centralized around the `assemble_pdf` function, which iterates through the Excel metadata and delegates rendering to specific page-type modules.

| Level 1  | Level 2          | Level 3               | Level 4             |
| :------- | :--------------- | :-------------------- | :------------------ |
| `main()` |                  |                       |                     |
|          | `assemble_pdf()` |                       |                     |
|          |                  | `register_fonts()`    |                     |
|          |                  | `is_clean_page()`     |                     |
|          |                  | `draw_page_chrome()`  |                     |
|          |                  |                       | `draw_footer()`     |
|          |                  |                       | `safe_draw_image()` |
|          |                  | `render_text_page()`  |                     |
|          |                  | `draw_source_text()`  |                     |
|          |                  | `render_table_page()` |                     |
|          |                  |                       | `_draw_rl_table()`  |
|          |                  | `render_chart_page()` |                     |
|          |                  |                       | `safe_draw_image()` |
|          |                  | `add_back_page()`     |                     |

---

## D. Function Documentation

_(Note: These functions have been documented based on their bespoke implementations for this specific reporting pipeline. They should be appended to the shared `ToolDescription_14_07_2026.csv` file if applicable)._

### `register_fonts()`

- **Description:** Idempotent function that registers the Montserrat TTF variants (Regular, Bold, Light) with ReportLab's PDFMetrics.
- **Arguments:** None
- **Returns:** `None`

### `set_font()`

- **Description:** Sets the active canvas font, falling back to Montserrat-Regular if the requested font is missing.
- **Arguments:**
  - `canvas` (Canvas): The active ReportLab canvas.
  - `font_name` (str): The name of the registered font.
  - `font_size` (int): The size of the font in points.
- **Returns:** `None`

### `safe_str()`

- **Description:** Safely converts a pandas object to a stripped string, handling NaNs gracefully.
- **Arguments:**
  - `x` (Any): The object/value to convert.
- **Returns:** `str` - The stripped string, or an empty string if NaN.

### `is_clean_page()`

- **Description:** Determines if an Excel configuration row lacks any text elements (Title, Subtitle, Source Text, Descriptive Text), meaning the page should be rendered as a full-bleed image.
- **Arguments:**
  - `page_meta` (dict): The dictionary representation of a single row from the Excel config.
- **Returns:** `bool` - True if no text elements exist, False otherwise.

### `safe_draw_image()`

- **Description:** Draws an image stretched to fill the full allocated bounding box, filling most of the page.
- **Arguments:**
  - `canvas` (Canvas): The active ReportLab canvas.
  - `path` (str): Absolute or relative path to the image.
  - `x`, `y`, `w`, `h` (float): Coordinates and dimensions for the bounding box.
- **Returns:** `None`

### `draw_source_text()`

- **Description:** Draws the source text at the bottom of the page as a wrapping ReportLab Paragraph.
- **Arguments:**
  - `c` (Canvas): The active canvas.
  - `page_meta` (dict): Row dictionary from the Excel config.
- **Returns:** `float` - The vertical height (in points) consumed by the wrapped paragraph.

### `draw_footer()`

- **Description:** Draws the grey footer band, optionally including the page number and right-aligned footer text.
- **Arguments:**
  - `canvas` (Canvas): The active canvas.
  - `footer_text` (str): The text to align to the right of the footer.
  - `page_num` (int): The current page number.
  - `show_page_num` (bool): Toggle indicating whether to physically draw the page number string.
- **Returns:** `None`

### `draw_page_chrome()`

- **Description:** Draws all repeating page decorations: footer, circles, logo, title, subtitle, and divider line. Skips top elements if the page is designated as "clean".
- **Arguments:**
  - `c` (Canvas): The active canvas.
  - `page_meta` (dict): Row dictionary from the Excel config.
  - `page_num` (int): Current page number.
  - `paths` (dict): Dictionary of paths for logos and decorative circles.
  - `is_clean` (bool): True if the page has no text elements.
  - `show_page_num` (bool): True if the page number should be drawn.
- **Returns:** `None`

### `_draw_rl_table()`

- **Description:** Helper function to horizontally center and draw a styled ReportLab Table.
- **Arguments:**
  - `c` (Canvas): The active canvas.
  - `data` (list): 2D list of table data.
  - `y` (float): The vertical bottom coordinate for the table.
- **Returns:** `None`

### `render_text_page()`

- **Description:** Renders a text-only page (e.g. Executive Summary) by wrapping the descriptive text inside a vertically aligned frame.
- **Arguments:**
  - `c` (Canvas): The active canvas.
  - `page_meta` (dict): Row dictionary from the Excel config.
- **Returns:** `None`

### `render_table_page()`

- **Description:** Renders a CSV file as a native ReportLab table. Dynamically calculates rows per page based on the bottom margin (to avoid overlapping source text) and automatically generates continuation pages if the table overflows.
- **Arguments:**
  - `c` (Canvas): The active canvas.
  - `page_meta` (dict): Row dictionary from the Excel config.
  - `csv_base_name` (str): The filename of the CSV.
  - `chart_folder` (str): Path to the folder containing the CSVs.
  - `paths` (dict): Image paths for page chrome.
  - `page_counter` (list): Mutable integer list tracking the global page number.
  - `source_h` (float): Height of the source text block to avoid overlap.
  - `show_page_num` (bool): Toggle for page numbering on continuation pages.
- **Returns:** `None`
- **Exceptions:** Logs an error and draws red text if the CSV file is missing.

### `render_chart_page()`

- **Description:** Renders a chart image. If the page is "clean", it renders full-bleed. Otherwise, it calculates the chart's bottom boundary dynamically based on `source_h` to prevent clipping.
- **Arguments:**
  - `c` (Canvas): The active canvas.
  - `page_meta` (dict): Row dictionary from the Excel config.
  - `chart_folder` (str): Path to the folder containing charts.
  - `source_h` (float): Height of the source text block.
  - `is_clean` (bool): If True, chart is drawn full-bleed.
  - `show_page_num` (bool): If False and is_clean is True, footer is omitted.
- **Returns:** `None`
- **Exceptions:** Logs an error if image rendering fails.

### `add_back_page()`

- **Description:** Renders the final contact and disclaimer page with a 2x2 contact grid and centered logo.
- **Arguments:**
  - `c` (Canvas): The active canvas.
  - `config` (dict): The main JSON configuration.
  - `first_row` (dict): The first row of the Excel metadata (containing contact details).
- **Returns:** `None`

### `assemble_pdf()`

- **Description:** The core orchestration function. Reads the JSON config and Excel metadata, loops through each row, and delegates to the appropriate `render_*` function based on the target content.
- **Arguments:**
  - `config` (dict): The main JSON configuration payload.
  - `uuid` (str): Unique identifier for naming the output PDF.
- **Returns:** `str` - The absolute path to the generated PDF.

### `main()`

- **Description:** Application entry point. Parses CLI arguments, loads the JSON config, and triggers `assemble_pdf`.
- **Arguments:**
  - `config_file_path` (str): CLI arg 1.
  - `uuid` (str): CLI arg 2.
- **Returns:** `None`

---

```

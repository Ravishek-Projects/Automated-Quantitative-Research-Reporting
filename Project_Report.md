# Project Report: FidelFolio PDF Report Generator

## Executive Summary

The goal of this project was to extract the PDF assembly logic from a monolithic reporting script into a dedicated, robust, and highly configurable module (`create_report_v2.py`). The focus was on improving pagination, resolving text/image overlap issues, and allowing flexible layout configurations (such as "clean" full-page images) directly from a centralized Excel configuration file.

## Development Journey

### Initial Constraints & Problems

Initially, the PDF generation was tightly coupled with chart creation using Matplotlib. CSV tables were rasterized as images, leading to significant limitations:

- **Pagination Issues:** Long tables generated from CSV files could not seamlessly paginate across multiple PDF pages.
- **Text Wrapping & Overlap:** Long source texts or descriptive blocks would push outside their bounds or cause the table image above them to overlap awkwardly.
- **Image Cropping:** Using standard aspect ratio preservation forced charts to be small and centered, leaving large patches of white space, while omitting it risked image cropping or stretching.

### Improvements: The Transition to ReportLab

The immediate architectural decision was to utilize the **ReportLab** library to render true text vectors and native tables.

- **Native Tables:** We built a detection system that identifies when the Excel configuration asks for a CSV file. Instead of pasting an image, the script natively reads the CSV via `pandas` and pipes it into a `ReportLab` table.
- **Dynamic Text Wrapping:** We replaced simple string drawing with `Paragraph` elements wrapped inside a `KeepInFrame`. This ensures that long text fields (such as Source Text or Descriptive Text) automatically wrap to the next line.

### Dynamic Spacing & Automation

- **Automatic CSV Detection:** The script was upgraded to automatically render _any_ `.csv` string in the "Chart" column as a native table, removing the need for manual `-drawtable` suffixes in the config.
- **Preserve Aspect Ratio overrides:** We set `preserveAspectRatio=False` on images, ensuring that chart images expand to fill the full allocated bounding box between the header chrome and footer.
- **Dynamic Table Margins:** We encountered an issue where extremely long source text would be overlapped by the bottom of the table. We resolved this by calculating the exact `source_h` (the height of the wrapped paragraph) _before_ rendering the table, allowing us to set a dynamic bottom margin that strictly prevents overlap.

### Final Implementation (v2 architecture)

The final iteration of the script introduced conditional layout toggles, creating a flawless, automated report assembly line:

1. **Dynamic Row Count:** Table pages now dynamically calculate how many rows fit on a page based on the remaining vertical space. The first page intelligently accounts for the source text height, while continuation pages default to a standard margin.
2. **Clean Pages:** An `is_clean_page` flag detects if an Excel row is completely devoid of text. If true, it strips the top page chrome (logo, title, circles) and expands the chart bounding box to the very top edge.
3. **Show Page Number Toggle:** A new column was added to the Excel input allowing page numbers to be hidden (`No`). If a clean page also hides the page number, the script removes the footer entirely, creating a true, 100% full-bleed image page.

## Conclusion

The `create_report_v2.py` script now serves as a robust, decoupled, and standalone presentation layer. It effectively translates business logic defined in an Excel sheet into a highly polished, paginated PDF report with zero graphical overlap or textual clipping.

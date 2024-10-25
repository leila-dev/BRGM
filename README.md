# BRGM
import os
from pypdf import PdfReader
import pdfplumber  # type: ignore
import pandas as pd

# Load the PDF file using PdfReader to get the number of pages
pdf_path = 'C:\\Users\\user\\OneDrive\\Desktop\\projet tal\\mayotte.pdf'
reader = PdfReader(pdf_path)
print(f'Number of pages: {len(reader.pages)}')

# Define the path for the output HTML file
html_path = 'C:\\Users\\user\\OneDrive\\Desktop\\projet tal\\tables_output.html'

# Remove the existing output file if it exists
if os.path.exists(html_path):
    try:
        os.remove(html_path)
    except Exception as e:
        print(f'Error deleting existing file: {e}')

# Open the PDF file using pdfplumber
with pdfplumber.open(pdf_path) as pdf:
    with open(html_path, 'w', encoding='utf-8') as html_file:
        # Write the table header for the HTML file
        html_file.write('<html><body>\n')
        html_file.write('<h1>Extracted Tables from PDF</h1>\n')

        # Iterate through each page of the PDF
        for page_number, page in enumerate(pdf.pages):
            # Extract tables from the current page
            tables = page.extract_tables()

            # Check if tables were found
            if tables:
                for idx, table in enumerate(tables):
                    # Validate if the extracted data resembles a table
                    if len(table) > 1 and all(len(row) > 1 for row in table):
                        # Create a DataFrame from the extracted table data
                        df = pd.DataFrame(table[1:], columns=table[0])  

                        # Validate DataFrame structure
                        if isinstance(df, pd.DataFrame) and not df.empty and df.shape[1] > 1:
                            # Write the table into HTML format
                            html_file.write(f'<h2>Page {page_number + 1} - Table {idx + 1}</h2>\n')
                            html_file.write(df.to_html(index=False, border=1))
                            html_file.write('<br>\n')  # Add space between tables

        # Write the closing tags for the HTML file
        html_file.write('</body></html>\n')

    print(f'Tables extracted successfully to {html_path}')

# File-Metadata-Extractor
This script can extract metadata from files to help investigators understand file properties such as creation date, modification date, author, etc. Metadata can often reveal crucial information about the file's origin, manipulation, and history.
import os
import magic
from datetime import datetime
from PIL import Image
from PyPDF2 import PdfFileReader

def get_file_metadata(file_path):
    # Check if the file exists
    if not os.path.isfile(file_path):
        print(f"File {file_path} does not exist.")
        return

    # Get basic file information
    file_info = {
        "File Name": os.path.basename(file_path),
        "File Size (bytes)": os.path.getsize(file_path),
        "File Type": magic.Magic(mime=True).from_file(file_path),
        "Creation Time": datetime.utcfromtimestamp(os.path.getctime(file_path)),
        "Modification Time": datetime.utcfromtimestamp(os.path.getmtime(file_path)),
        "Last Accessed": datetime.utcfromtimestamp(os.path.getatime(file_path)),
    }

    print("\nFile Metadata:\n")
    for key, value in file_info.items():
        print(f"{key}: {value}")

    # Additional metadata extraction for specific file types
    if file_info["File Type"].startswith("image"):
        extract_image_metadata(file_path)
    elif file_info["File Type"] == "application/pdf":
        extract_pdf_metadata(file_path)

def extract_image_metadata(image_path):
    try:
        with Image.open(image_path) as img:
            exif_data = img._getexif()
            print("\nImage Metadata:")
            if exif_data:
                for tag, value in exif_data.items():
                    print(f"{tag}: {value}")
            else:
                print("No EXIF data found.")
    except Exception as e:
        print(f"Error extracting image metadata: {e}")

def extract_pdf_metadata(pdf_path):
    try:
        with open(pdf_path, "rb") as file:
            pdf_reader = PdfFileReader(file)
            info = pdf_reader.getDocumentInfo()
            print("\nPDF Metadata:")
            for key, value in info.items():
                print(f"{key}: {value}")
    except Exception as e:
        print(f"Error extracting PDF metadata: {e}")

# Example usage
if __name__ == "__main__":
    file_path = input("Enter the path of the file to analyze: ")
    get_file_metadata(file_path)

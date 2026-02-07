# soft-nexis-task-1
File Organizer Script  This script organizes files in a specified directory (and its subdirectories) by moving them into category folders based on their file extensions. Categories include Code, Documents, Images, Videos, Audio, Archives, and Others (
import os
import shutil
import logging
import argparse

# Set up logging
logging.basicConfig(
    filename='file_organizer.log',
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

# Define extension to category mapping
EXTENSION_CATEGORIES = {
    'Code': ['.py', '.java', '.cpp', '.c', '.js', '.html', '.css'],
    'Documents': ['.txt', '.doc', '.docx', '.pdf', '.rtf'],
    'Images': ['.jpg', '.jpeg', '.png', '.gif', '.bmp', '.tiff'],
    'Videos': ['.mp4', '.avi', '.mkv', '.mov'],
    'Audio': ['.mp3', '.wav', '.flac'],
    'Archives': ['.zip', '.rar', '.tar', '.gz'],
    'Others': []  # Default for unrecognized extensions
}

def get_category(extension):
    """Determine the category based on file extension."""
    for category, extensions in EXTENSION_CATEGORIES.items():
        if extension.lower() in extensions:
            return category
    return 'Others'

def resolve_duplicate(target_dir, filename):
    """Resolve duplicate filenames by appending a number."""
    base, ext = os.path.splitext(filename)
    counter = 1
    new_filename = filename
    while os.path.exists(os.path.join(target_dir, new_filename)):
        new_filename = f"{base}_{counter}{ext}"
        counter += 1
    return new_filename

def organize_files(source_dir):
    """Scan and organize files in the specified directory."""
    if not os.path.exists(source_dir):
        logging.error(f"Source directory '{source_dir}' does not exist.")
        return

    for root, dirs, files in os.walk(source_dir):
        for file in files:
            file_path = os.path.join(root, file)
            _, extension = os.path.splitext(file)
            category = get_category(extension)
            
            # Create category folder if it doesn't exist
            category_dir = os.path.join(source_dir, category)
            try:
                os.makedirs(category_dir, exist_ok=True)
                logging.info(f"Created or confirmed directory: {category_dir}")
            except OSError as e:
                logging.error(f"Failed to create directory '{category_dir}': {e}")
                continue
            
            # Determine target path and handle duplicates
            target_filename = resolve_duplicate(category_dir, file)
            target_path = os.path.join(category_dir, target_filename)
            
            # Move the file
            try:
                shutil.move(file_path, target_path)
                logging.info(f"Moved '{file_path}' to '{target_path}'")
            except (OSError, PermissionError) as e:
                logging.error(f"Failed to move '{file_path}' to '{target_path}': {e}")

def main():
    parser = argparse.ArgumentParser(description="Organize files in a directory by extension.")
    parser.add_argument('directory', help="Path to the directory to organize.")
    args = parser.parse_args()
    
    logging.info(f"Starting file organization in '{args.directory}'")
    organize_files(args.directory)
    logging.info("File organization completed.")

if __name__ == "__main__":
    main()

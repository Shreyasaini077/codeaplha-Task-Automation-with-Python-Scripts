"""
🖼️ File Organizer Pro - Smart Image Sorter
Created with ❤️ by Python Enthusiast
A sophisticated tool that:
- Organizes images by year/month based on creation date
- Handles duplicate filenames intelligently
- Preserves original file attributes
- Creates detailed logs of all operations
"""
import os
import shutil
import time
from datetime import datetime
import hashlib
import logging
from PIL import Image
from PIL.ExifTags import TAGS

class ImageOrganizer:
    def __init__(self, source_dir="~/Downloads", dest_dir="~/Pictures"):
        self.source_dir = os.path.expanduser(source_dir)
        self.dest_dir = os.path.expanduser(dest_dir)
        self.log_file = os.path.join(os.path.dirname(__file__), "image_organizer.log")
        self.setup_logging()
        self.file_count = 0
        self.duplicates_found = 0
        self.image_extensions = ('.jpg', '.jpeg', '.png', '.gif', '.bmp', '.tiff', '.webp')
        self.COLOR = {
            'HEADER': '\033[95m',
            'SUCCESS': '\033[92m',
            'WARNING': '\033[93m',
            'ERROR': '\033[91m',
            'ENDC': '\033[0m',
        }
    def setup_logging(self):
        """Configure detailed logging for all operations"""
        logging.basicConfig(
            filename=self.log_file,
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s',
            datefmt='%Y-%m-%d %H:%M:%S'
        )
        logging.info(f"Initializing Image Organizer")
        logging.info(f"Source directory: {self.source_dir}")
        logging.info(f"Destination directory: {self.dest_dir}")
    def print_with_color(self, message, color='HEADER'):
        """Print colored messages to console"""
        print(f"{self.COLOR[color]}{message}{self.COLOR['ENDC']}")
    def get_file_hash(self, filepath):
        """Generate MD5 hash of file content for duplicate detection"""
        hash_md5 = hashlib.md5()
        with open(filepath, "rb") as f:
            for chunk in iter(lambda: f.read(4096), b""):
                hash_md5.update(chunk)
        return hash_md5.hexdigest()
    def get_image_date(self, filepath):
        """
        Try to extract date from EXIF data, fall back to creation date
        Returns: datetime object or None
        """
        try:
            with Image.open(filepath) as img:
                exif_data = img._getexif()
                if exif_data:
                    for tag, value in exif_data.items():
                        decoded = TAGS.get(tag, tag)
                        if decoded == 'DateTimeOriginal':
                            return datetime.strptime(value, "%Y:%m:%d %H:%M:%S")
        except Exception as e:
            logging.warning(f"Could not read EXIF data from {filepath}: {str(e)}")
        try:
            timestamp = os.path.getmtime(filepath)
            return datetime.fromtimestamp(timestamp)
        except Exception as e:
            logging.error(f"Could not get creation date for {filepath}: {str(e)}")
            return None
    def organize_images(self):
        """Main organization logic"""
        self.print_with_color("\n🔍 Starting image organization process...", 'HEADER')
        logging.info("Starting organization process")
        if not os.path.exists(self.source_dir):
            self.print_with_color(f"❌ Source directory not found: {self.source_dir}", 'ERROR')
            logging.error(f"Source directory not found: {self.source_dir}")
            return
        os.makedirs(self.dest_dir, exist_ok=True)
        for root, _, files in os.walk(self.source_dir):
            for filename in files:
                if filename.lower().endswith(self.image_extensions):
                    source_path = os.path.join(root, filename)
                    self.process_image(source_path)
        self.print_with_color("\n📝 Summary:", 'HEADER')
        self.print_with_color(f"• Total files processed: {self.file_count}", 'SUCCESS')
        self.print_with_color(f"• Duplicates handled: {self.duplicates_found}", 'WARNING')
        self.print_with_color(f"• Detailed log available at: {self.log_file}", 'HEADER')
        logging.info(f"Organization complete. Processed {self.file_count} files.")
    def process_image(self, source_path):
        """Process individual image file"""
        self.file_count += 1
        logging.info(f"Processing file: {source_path}")
        file_date = self.get_image_date(source_path)
        if not file_date:
            logging.warning(f"Skipping file (could not determine date): {source_path}")
            self.print_with_color(f"⚠️ Could not determine date for: {os.path.basename(source_path)}", 'WARNING')
            return
        year = file_date.strftime("%Y")
        month = file_date.strftime("%m-%B")
        dest_folder = os.path.join(self.dest_dir, year, month)
        os.makedirs(dest_folder, exist_ok=True)
        dest_path = os.path.join(dest_folder, os.path.basename(source_path))
        if os.path.exists(dest_path):
            if self.get_file_hash(source_path) == self.get_file_hash(dest_path):
                logging.info(f"Exact duplicate found, skipping: {source_path}")
                self.duplicates_found += 1
                self.print_with_color(f"♻️ Duplicate skipped: {os.path.basename(source_path)}", 'WARNING')
                return
            else:
                base, ext = os.path.splitext(os.path.basename(source_path))
                counter = 1
                while True:
                    new_filename = f"{base}_{counter}{ext}"
                    new_dest_path = os.path.join(dest_folder, new_filename)
                    if not os.path.exists(new_dest_path):
                        dest_path = new_dest_path
                        break
                    counter += 1
        try:
            shutil.copy2(source_path, dest_path)
            logging.info(f"Moved {source_path} to {dest_path}")
            self.print_with_color(f"✓ Organized: {os.path.basename(source_path)} → {year}/{month}/", 'SUCCESS')
        except Exception as e:
            logging.error(f"Error moving file {source_path}: {str(e)}")
            self.print_with_color(f"❌ Error moving {os.path.basename(source_path)}: {str(e)}", 'ERROR')

if __name__ == "__main__":
    organizer = ImageOrganizer()
    print(r"""
    ____ ___  ____  __  ______   __________      ______ __________ 
   / __ |__ \/ __ \/ / / / __/  / ____/ __ \    / ____// ____/ __ \
  / / / __/ / / / / / / / /_   / __/ / /_/ /   / __/  / /   / / / /
 / /_/ / __/ /_/ / /_/ / __/  / /___/ _, _/   / /___ / /___/ /_/ / 
 \____/____/\____/\____/_/    /_____/_/ |_|   /_____/ \____/_____/  
    """)
    organizer.organize_files()


# ChinaCars - Data Processing
![car-design-hologram](car-design-hologram.jpg)

# Overview
This project involves a set of scripts that clean, translate, and process a dataset of car owners from China. The dataset is assumed to contain columns in Chinese, which are translated into English before undergoing further data cleaning, validation, and restructuring.

## Requirements
- **Pandas:** Used for data manipulation and analysis.
- **NumPy:** Utilized for array handling and splitting the dataset into chunks.
- **OS:** Used to interact with the file system.

## Features
- **Language Detection and Translation:** Automatically detects the language of column headers and translates them to English using Google Translate.
- **Data Cleaning and Merging:** Merges address-related columns into one, removes duplicates, and validates email formats.
- **Data Chunking:** The large dataset is divided into smaller, manageable chunks for easier processing.
- **Garbage Collection:** Invalid or incomplete rows are saved into a separate "garbage" file for review.

## Installation

1. Install required Python libraries:
    ```bash
    pip install pandas googletrans==4.0.0-rc1 langdetect numpy
    ```
This function detects the language used in the CSV and then proceeds to translate the file. 

## Usage

### 1. Translation of Column Headers

This script reads a CSV file, detects the language of the column headers, and translates non-English headers to English:

```python
import pandas as pd
from googletrans import Translator
from langdetect import detect

def translate_headers(df):
    translator = Translator()
    new_headers = []
    for header in df.columns:
        detected_lang = detect(header)
        if detected_lang != 'en':
            translation = translator.translate(header, dest='en')
            new_headers.append(translation.text)
        else:
            new_headers.append(header)
    return new_headers

# Usage:
df = pd.read_csv('/path/to/input.csv')
df.columns = translate_headers(df)
df.to_csv('/path/to/output.csv', index=False)
```

### 2. Merging Address Fields and Removing Duplicates

This function merges province, city, and address into a single column, removes duplicate rows, and outputs a cleaned DataFrame:

```python
import pandas as pd

def merge_and_drop_duplicates(file_path):
    df = pd.read_csv(file_path)
    df['address'] = df['Province'] + ', ' + df['City'] + ', ' + df['address']
    df = df.drop(columns=['Province', 'City']).drop_duplicates()
    return df

# Usage:
df = merge_and_drop_duplicates('/path/to/input.csv')
df.to_csv('/path/to/output_clean.csv', index=False)
```

### 3. Email Validation and Data Chunking

This function validates email addresses, cleans the data, and splits it into 4 chunks. Invalid and incomplete rows are saved to a separate garbage file:

```python
import pandas as pd
import numpy as np
import re

def is_valid_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None

def chunk_and_clean_data(file_path):
    df = pd.read_csv(file_path)
    df['Invalid Email'] = df['Mail'].apply(lambda x: not is_valid_email(x))
    
    df['full_address'] = df['Province'] + ', ' + df['City'] + ', ' + df['address']
    df = df.drop(columns=['Province', 'City', 'address'])

    chunks = np.array_split(df, 4)
    for i, chunk in enumerate(chunks):
        chunk.to_csv(f'cleaned_chunk_{i+1}.csv', index=False)

    garbage_df = df[df['Invalid Email'] | df.isnull().any(axis=1)]
    garbage_df.to_csv('garbage_data.csv', index=False)

# Usage:
chunk_and_clean_data('/path/to/input.csv')
```

## Example File Structure

```
/ChinaCars
│
├── main.py            # Main script containing functions
├── requirements.txt   # List of required Python packages
├── README.md          # Project documentation (this file)
└── data
    ├── input.csv      # Original dataset
    ├── output.csv     # Translated dataset
    ├── cleaned_chunk_1.csv  # Cleaned chunk 1
    ├── cleaned_chunk_2.csv  # Cleaned chunk 2
    └── garbage_data.csv     # Invalid rows
```

## Notes
- Ensure your dataset contains the necessary columns (`Province`, `City`, `Mail`, etc.) before running the script.
- The email validation is basic and can be extended to include more complex rules.

## License

This project is licensed under the MIT License.

--- 

This README outlines the purpose, functionality, and usage of the scripts in the project. You can customize the paths and add more details as needed for your GitHub repository.

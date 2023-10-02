# Working with python
## Extract Data from JSONL to Excel 
This Python script extracts data from a JSONL file and creates an Excel file with selected attributes. It uses the argparse library for command-line argument handling and the pandas library for working with data in DataFrames.

- Imports necessary libraries: json, os, pandas (as pd), and argparse.
- Creates an ArgumentParser object (parser) to parse command-line arguments.
- Defines several command-line arguments such as input directory (--input-dir), output directory (--output-dir), language code (--language), and a verbose mode flag (--verbose).
- Parses the command-line arguments and stores their values in corresponding variables (input_dir, output_dir, language, verbose).
- Creates the output directory (output_dir) if it doesn't already exist.
- Iterates through each file in the input directory (input_dir) and selects files with a .jsonl extension.
- For each selected JSONL file, it reads the file, extracts specific attributes (id, utt, annot_utt) from the JSON data, and filters records based on the specified language code (language).
- We then creates a Pandas DataFrame (selected_data) to store the filtered data.
- Determines the output filename based on the input JSONL filename, appends the language code, and changes the extension to .xlsx.
- Writes the selected and filtered data to an Excel file in the output directory.

### main.py
```python
import json
import os
import pandas as pd
import argparse

# Create an ArgumentParser object
parser = argparse.ArgumentParser(description='Process JSONL files and save relevant data to Excel files.')

# Add flags and arguments
parser.add_argument('--input-dir', '-i', type=str, default='./data/dataset', help='Directory containing JSONL files')
parser.add_argument('--output-dir', '-o', type=str, default='./outputs', help='Output directory for Excel files')
parser.add_argument('--language', '-l', type=str, default='en', help='Language code to filter JSONL files')
parser.add_argument('--verbose', '-v', action='store_true', help='Enable verbose mode')

# Parse the command-line arguments
args = parser.parse_args()

# Access the values of the flags and arguments
input_dir = args.input_dir
output_dir = args.output_dir
language = args.language
verbose = args.verbose

# Create the output folder if it doesn't exist
os.makedirs(output_dir, exist_ok=True)

# Loop through each JSONL file in the directory
for filename in os.listdir(input_dir):
    if filename.endswith('.jsonl'):
        file_path = os.path.join(input_dir, filename)

        # Create an empty DataFrame to store the relevant data
        selected_data = pd.DataFrame(columns=['id', 'utt', 'annot_utt'])

        # Read the JSONL file and extract relevant attributes
        with open(file_path, 'r') as json_file:
            data = [json.loads(line) for line in json_file]
            for record in data:
                # Extract the language code from the "locale" attribute
                locale = record.get('locale', '')
                language_code = locale.split('-')[0] if locale else ''

                # Filter records for the specified language code
                if language_code == language:
                    selected_data = pd.concat([selected_data, pd.DataFrame({
                        'id': [record.get('id', '')],
                        'utt': [record.get('utt', '')],
                        'annot_utt': [record.get('annot_utt', '')]
                    })], ignore_index=True)

        # Determine the output filename based on the input JSONL filename
        output_filename = os.path.splitext(language + filename)[0] + '.xlsx'
        output_file = os.path.join(output_dir, output_filename)

        # Write the selected data to an Excel file
        selected_data.to_excel(output_file, index=False)

        if verbose:
            print(f"Processed {filename} and saved as {output_filename}")

print("Excel files generated for each JSONL file and stored in the 'outputs'")
```
### generator.sh
- This bash file works along with the main.py file
- Sets default values for the input_dir and output_dir variables.
- Defines flags (-i and -o) and their default values using the getopts command.
- Iterates through each .jsonl file in the specified input directory.
- For each JSONL file, it determines the output filename by changing the extension to .xlsx.
- Calls the Python script (main.py) with flags to process the JSONL file and create an Excel file.
- Prints a message indicating that it processed the JSONL file and created the corresponding Excel file.

```bash
#!/bin/bash

# Default values for flags
input_dir='dataset/data_files'
output_dir='../outputs'

# Define flags and their default values
while getopts "i:o:" opt; do
    case $opt in
        i) input_dir="$OPTARG";;
        o) output_dir="$OPTARG";;
        \?) echo "Invalid option: -$OPTARG" >&2; exit 1;;
    esac
done

# Loop through each JSONL file in the input directory
for filename in "$input_dir"/*.jsonl; do
    if [ -f "$filename" ]; then
        # Determine the output filename based on the input JSONL filename
        output_filename=$(basename -- "$filename" .jsonl).xlsx
        output_file="$output_dir/$output_filename"

        # Call the Python script with flags to extract data and create an Excel file
        python main.py --input "$filename" --output "$output_file"

        echo "Processed $filename and created $output_file"
    fi
done

echo "Excel files generated for each JSONL file and stored in the 'outputs'"
```
In summary, the Bash script is responsible for iterating through JSONL files in a specified input directory, determining output filenames, and calling the Python script to process and convert the data to Excel format. The Python script, on the other hand, handles the actual data processing, filtering, and Excel file creation based on command-line arguments. Together, these scripts work as a pipeline to convert JSONL files into Excel files 

## Generating seperate files for English, Swahili and German
In this section we aim to generate separate jsonl file for test, train and dev for different languages

1. Define Input and Output Directories:
- It specifies the input directory (input_dir) containing the original JSONL files.
- It defines the output directory (output_dir) where separated JSONL files will be saved.
2. Create Output Directory:
- It creates the output directory specified in output_dir if it doesn't already exist using os.makedirs(output_dir, exist_ok=True).
3. Define Languages and Partitions:
- It lists the languages in the languages list (['en', 'sw', 'de']) and partitions in the partitions list (['test', 'train', 'dev']) that need to be processed.
4. Loop Through Languages and Partitions:
- It iterates through each language (lang) and partition (partition) in nested loops.
5. Define Output JSONL File:
- For each combination of language and partition, it defines the output JSONL file name based on the language and partition, e.g., 'en-test.jsonl'.
- It creates the full output file path by joining the output directory and the output file name.
6. Create an Empty Records List:
- It initializes an empty list called records to store the records that match the current language and partition.
7. Loop Through Original JSONL Files:
- It iterates through the original JSONL files in the input directory.
- For each JSONL file, it reads its content line by line.
8. Filter and Collect Records:
- It parses each line as JSON and checks if the data matches the current language and partition based on the "locale" and "partition" attributes.
- If a data record matches the language and partition criteria, it is appended to the records list.
9. Write Selected Records to Output JSONL File:
- It writes the selected records to the corresponding output JSONL file.
- For each record in the records list, it serializes it as JSON and writes it to the output file, ensuring that non-ASCII characters are handled properly.
10. Print Progress:
- After processing a specific language-partition combination, it prints a message indicating that the corresponding output JSONL file has been generated, e.g., 'Generated en-test.jsonl'.

### Separate.py
```python
Define Input and Output Directories:

    It specifies the input directory (input_dir) containing the original JSONL files.
    It defines the output directory (output_dir) where separated JSONL files will be saved.

Create Output Directory:

    It creates the output directory specified in output_dir if it doesn't already exist using os.makedirs(output_dir, exist_ok=True).

Define Languages and Partitions:

    It lists the languages in the languages list (['en', 'sw', 'de']) and partitions in the partitions list (['test', 'train', 'dev']) that need to be processed.

Loop Through Languages and Partitions:

    It iterates through each language (lang) and partition (partition) in nested loops.

Define Output JSONL File:

    For each combination of language and partition, it defines the output JSONL file name based on the language and partition, e.g., 'en-test.jsonl'.
    It creates the full output file path by joining the output directory and the output file name.

Create an Empty Records List:

    It initializes an empty list called records to store the records that match the current language and partition.

Loop Through Original JSONL Files:

    It iterates through the original JSONL files in the input directory.
    For each JSONL file, it reads its content line by line.

Filter and Collect Records:

    It parses each line as JSON and checks if the data matches the current language and partition based on the "locale" and "partition" attributes.
    If a data record matches the language and partition criteria, it is appended to the records list.

Write Selected Records to Output JSONL File:

    It writes the selected records to the corresponding output JSONL file.
    For each record in the records list, it serializes it as JSON and writes it to the output file, ensuring that non-ASCII characters are handled properly.
```

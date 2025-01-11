# Abdulrahman
Projects 
import pandas as pd
import numpy as np
import json
import requests
from bs4 import BeautifulSoup
from sklearn.preprocessing import StandardScaler, OneHotEncoder
import threading
from queue import Queue
import os

#loads csv data from file path:
def load_data(file_path):
    data = pd.read_csv(file_path)
    return data

#cleans data by dropping missing values and duplicates:
def clean_data(data):
    cleaned_data = data.dropna(subset=['location', 'total_cases', 'total_deaths'])
    cleaned_data = cleaned_data.drop_duplicates()
    return cleaned_data

#scales specified column using standard scaler:
def scale_data(data, column_name):
    scaler = StandardScaler()
    scaled_column = scaler.fit_transform(data[[column_name]])
    scaled_column = np.round(scaled_column, 2)
    return scaled_column

#encodes specified column using one hot encoder:
def encode_data(data, column_name):
    encoder = OneHotEncoder()
    encoded = encoder.fit_transform(data[[column_name]])
    return encoded.toarray()

#writes data to a json file:
def write_json(data, file_path):
#check if the directory exists, if not create it:
    directory = os.path.dirname(file_path)
    if not os.path.exists(directory):
        os.makedirs(directory)

#now write the JSON file:
    with open(file_path, 'w') as file:
        json.dump(data, file)

#reads data from a json file:
def read_json(file_path):
    with open(file_path, 'r') as file:
        return json.load(file)

#scrapes data from a webpage by extracting all h2 elements:
def web_scrape(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.content, "html.parser")
    return [element.text for element in soup.find_all('h2')]

#processes data chunk by summing numeric columns:
def process_chunk(chunk):
    numeric_chunk = chunk.select_dtypes(include=[np.number])
    return numeric_chunk.sum().values

#runs multithreading for processing data in chunks:
def run_multithreading(data_chunks):
    threads = []
    results = Queue()
    def thread_function(result_queue, chunk):
        result_queue.put(process_chunk(chunk))
    for chunk in data_chunks:
        thread = threading.Thread(target=thread_function, args=(results, chunk))
        threads.append(thread)
        thread.start()
    for thread in threads:
        thread.join()

    #collect all results from the queue:
    all_results = []
    while not results.empty():
        all_results.append(results.get())
    return np.sum(all_results, axis=0)

#calculates total deaths and highest number of deaths:
def calculate_deaths(data):
    if 'total_deaths' not in data.columns or data.empty:
        return 0, 0
    total_deaths = data['total_deaths'].sum()
    max_deaths = data['total_deaths'].max()
    return total_deaths, max_deaths

#main function to run the full pipeline:
def main():
    file_path = r"owid-covid-data.csv"
    data = pd.read_csv(file_path)
    cleaned_data = clean_data(data)
    cleaned_data['scaled_cases'] = scale_data(cleaned_data, 'total_cases')
    encoded_countries = encode_data(cleaned_data, 'location')
    json_data = {"example": {"key": "value", "nested": {"number": 42}}}
    json_file_path = './projects/example.json'
    write_json(json_data, json_file_path)
    loaded_json = read_json(json_file_path)
    nested_value = loaded_json["example"]["nested"]["number"]
    url = "https://example.com"
    scraped_data = web_scrape(url)

    #split data for multithreading:
    data_chunks = [
        cleaned_data.iloc[:len(cleaned_data) // 2],
        cleaned_data.iloc[len(cleaned_data) // 2:],]
    result = run_multithreading(data_chunks)

    #calculate death statistics
    total_deaths, max_deaths = calculate_deaths(cleaned_data)
    print(cleaned_data.head())
    print(encoded_countries[:5])
    print("Nested value from JSON:", nested_value)
    print("Scraped data:", scraped_data)
    print("Processed data sum:", result)
    print("Total deaths:", total_deaths)
    print("Highest number of deaths:", max_deaths)
if __name__ == "__main__":
    main()

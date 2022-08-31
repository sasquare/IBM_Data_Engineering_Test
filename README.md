# IBM_Data_Engineering_Test
Peer to Peer Review Assignment - Data Engineer - ETL

Peer Review Assignment - Data Engineer - ETL
Estimated time needed: 20 minutes

Objectives
In this final part you will:

Run the ETL process
Extract bank and market cap data from the JSON file bank_market_cap.json
Transform the market cap currency using the exchange rate data
Load the transformed data into a seperate CSV
For this lab, we are going to be using Python and several Python libraries. Some of these libraries might be installed in your lab environment or in SN Labs. Others may need to be installed by you. The cells below will install these libraries when executed.

#!mamba install pandas==1.3.3 -y
#!mamba install requests==2.26.0 -y
Imports
Import any additional libraries you may need here.

import glob
import pandas as pd
from datetime import datetime
As the exchange rate fluctuates, we will download the same dataset to make marking simpler. This will be in the same format as the dataset you used in the last section

!wget https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0221EN-SkillsNetwork/labs/module%206/Lab%20-%20Extract%20Transform%20Load/data/bank_market_cap_1.json
!wget https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0221EN-SkillsNetwork/labs/module%206/Lab%20-%20Extract%20Transform%20Load/data/bank_market_cap_2.json
!wget https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0221EN-SkillsNetwork/labs/module%206/Final%20Assignment/exchange_rates.csv
--2022-08-31 06:54:20--  https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0221EN-SkillsNetwork/labs/module%206/Lab%20-%20Extract%20Transform%20Load/data/bank_market_cap_1.json
Resolving cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud (cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud)... 169.63.118.104
Connecting to cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud (cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud)|169.63.118.104|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2815 (2.7K) [application/json]
Saving to: ‘bank_market_cap_1.json’

bank_market_cap_1.j 100%[===================>]   2.75K  --.-KB/s    in 0s      

2022-08-31 06:54:21 (20.7 MB/s) - ‘bank_market_cap_1.json’ saved [2815/2815]

--2022-08-31 06:54:22--  https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0221EN-SkillsNetwork/labs/module%206/Lab%20-%20Extract%20Transform%20Load/data/bank_market_cap_2.json
Resolving cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud (cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud)... 169.63.118.104
Connecting to cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud (cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud)|169.63.118.104|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1429 (1.4K) [application/json]
Saving to: ‘bank_market_cap_2.json’

bank_market_cap_2.j 100%[===================>]   1.40K  --.-KB/s    in 0s      

2022-08-31 06:54:23 (12.1 MB/s) - ‘bank_market_cap_2.json’ saved [1429/1429]

--2022-08-31 06:54:23--  https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-PY0221EN-SkillsNetwork/labs/module%206/Final%20Assignment/exchange_rates.csv
Resolving cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud (cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud)... 169.63.118.104
Connecting to cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud (cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud)|169.63.118.104|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 590 [text/csv]
Saving to: ‘exchange_rates.csv’

exchange_rates.csv  100%[===================>]     590  --.-KB/s    in 0s      

2022-08-31 06:54:24 (8.75 MB/s) - ‘exchange_rates.csv’ saved [590/590]

Extract
JSON Extract Function
This function will extract JSON files.

def extract_from_json(file_to_process):
    dataframe = pd.read_json(file_to_process)
    return dataframe
Extract Function
Define the extract function that finds JSON file bank_market_cap_1.json and calls the function created above to extract data from them. Store the data in a pandas dataframe. Use the following list for the columns.

columns=['Name','Market Cap (US$ Billion)']
market_cap_file = 'bank_market_cap_1.json'
exchange_rate_file = 'exchange_rates.csv'
load_to_file = 'bank_market_cap_gbp.csv'
def extract(file_name):
    extracted_data = extract_from_json(file_name)
    df = pd.DataFrame(extracted_data, columns=columns)
    
    return df
Question 1 Load the file exchange_rates.csv as a dataframe and find the exchange rate for British pounds with the symbol GBP, store it in the variable exchange_rate, you will be asked for the number. Hint: set the parameter index_col to 0.

# Write your code here
rates = pd.read_csv(exchange_rate_file, index_col=0)
exchange_rate = rates.at['GBP', 'Rates']
exchange_rate
0.7323984208000001
Transform
Using exchange_rate and the exchange_rates.csv file find the exchange rate of USD to GBP. Write a transform function that

Changes the Market Cap (US$ Billion) column from USD to GBP
Rounds the Market Cap (US$ Billion)` column to 3 decimal places
Rename Market Cap (US$ Billion) to Market Cap (GBP$ Billion)
def transform(extracted_df, exchange_rate):
    transformed_df = extracted_df.rename(columns={'Market Cap (US$ Billion)': 'Dollar'})
    transformed_df['Dollar'] = round(transformed_df.Dollar * exchange_rate, 3)
    transformed_df = transformed_df.rename(columns={'Dollar': 'Market Cap (GBP$ Billion)'})
    
    return transformed_df
Load
Create a function that takes a dataframe and load it to a csv named bank_market_cap_gbp.csv. Make sure to set index to False.

def load(df_to_load, filename):
    df_to_load.to_csv(filename)
    
Logging Function
Write the logging function log to log your data:

def log(message):
    timestamp_format = '%d-%h-%Y-%H:%M:%S'
    time = datetime.now()
    timestamp = time.strftime(timestamp_format)
    with open("logfile.txt","a") as f:
        f.write(f'{timestamp}, {message}\n')
Running the ETL Process
Log the process accordingly using the following "ETL Job Started" and "Extract phase Started"

# Write your code here
log("ETL Job Started")
log("Extract Phase Started")
Extract
Question 2 Use the function extract, and print the first 5 rows, take a screen shot:

# Call the function here
extracted_data = extract(market_cap_file)
​
# Print the rows here
extracted_data.head()
Name	Market Cap (US$ Billion)
0	JPMorgan Chase	390.934
1	Industrial and Commercial Bank of China	345.214
2	Bank of America	325.331
3	Wells Fargo	308.013
4	China Construction Bank	257.399
Log the data as "Extract phase Ended"

# Write your code here
log('Extract Phase Ended')
Transform
Log the following "Transform phase Started"

# Write your code here
log('Transform Phase Started')
Question 3 Use the function transform and print the first 5 rows of the output, take a screen shot:

# Call the function here
transformed_data = transform(extracted_data, exchange_rate)
# Print the first 5 rows here
transformed_data.head()
Name	Market Cap (GBP$ Billion)
0	JPMorgan Chase	286.319
1	Industrial and Commercial Bank of China	252.834
2	Bank of America	238.272
3	Wells Fargo	225.588
4	China Construction Bank	188.519
Log your data "Transform phase Ended"

# Write your code here
log('Transform Phase Ended')
Load
Log the following "Load phase Started".

# Write your code here
log('Load Phase Started')
Call the load function

# Write your code here
load(transformed_data, load_to_file)
Log the following "Load phase Ended".

# Write your code here
log('Load Phase Ended')
# Write your code here
log('Load Phase Ended')

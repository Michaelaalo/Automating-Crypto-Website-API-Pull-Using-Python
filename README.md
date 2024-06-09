# Automating-Crypto-Website-API-Pull-Using-Python
## Cryptocurrency Price Analysis

This Jupyter Notebook demonstrates how to use the CoinMarketCap API to gather and analyze cryptocurrency data. The code is structured to make API calls, process the retrieved data, and visualize trends. Below is an explanation of each section of the code.

## Setup

First, we import necessary libraries and set up the CoinMarketCap API URL and parameters:

```python
from requests import Request, Session
from requests.exceptions import ConnectionError, Timeout, TooManyRedirects
import json
import pandas as pd

url = 'https://pro-api.coinmarketcap.com/v1/cryptocurrency/listings/latest'
parameters = {
  'start': '1',
  'limit': '15',
  'convert': 'USD'
}
headers = {
  'Accepts': 'application/json',
  'X-CMC_PRO_API_KEY': '0ad53085-1cb2-4eb8-ad9e-3ffbd7e56509',
}

session = Session()
session.headers.update(headers)
```

We import the `requests` library to handle HTTP requests and `pandas` for data manipulation.

## API Request

Next, we define a function to make the API call and handle potential errors:

```python
try:
  response = session.get(url, params=parameters)
  data = json.loads(response.text)
except (ConnectionError, Timeout, TooManyRedirects) as e:
  print(e)
```

## Data Normalization

We normalize the JSON data into a pandas DataFrame and add a timestamp:

```python
df = pd.json_normalize(data['data'])
df['timestamp'] = pd.to_datetime('now')
df
```

## Running the API Call Periodically

The `api_runner` function is defined to automate the API calls and append the data to the DataFrame:

```python
def api_runner():
    global df
    session = Session()
    session.headers.update(headers)

    try:
      response = session.get(url, params=parameters)
      data = json.loads(response.text)
    except (ConnectionError, Timeout, TooManyRedirects) as e:
      print(e)

    df2 = pd.json_normalize(data['data'])
    df2['Timestamp'] = pd.to_datetime('now')
    df = df.append(df2)
```

We run this function every minute for a certain number of iterations:

```python
import os 
from time import time
from time import sleep

for i in range(333):
    api_runner()
    print('API Runner completed')
    sleep(60) #sleep for 1 minute
exit()
```

## Data Visualization

### Cryptocurrency Trends

We analyze the percentage changes of cryptocurrencies over different periods:

```python
df3 = df.groupby('name', sort=False)[['quote.USD.percent_change_1h', 'quote.USD.percent_change_24h', 'quote.USD.percent_change_7d', 'quote.USD.percent_change_30d', 'quote.USD.percent_change_60d', 'quote.USD.percent_change_90d']].mean()
df4 = df3.stack()
df5 = df4.to_frame(name='values')
df5.count()
```

We then visualize these trends using Seaborn:

```python
import seaborn as sns
import matplotlib.pyplot as plt

sns.catplot(x='percent_change', y='values', hue='name', data=df7, kind='point')
```

### Bitcoin Price Over Time

We create a simpler DataFrame focusing on Bitcoin's price and plot it:

```python
df10 = df[['name', 'quote.USD.price', 'timestamp']]
df10 = df10.query("name == 'Bitcoin'")
sns.set_theme(style="darkgrid")
sns.lineplot(x='timestamp', y='quote.USD.price', data = df10)
```

## Conclusion

This notebook provides a basic framework for retrieving, processing, and visualizing cryptocurrency data from the CoinMarketCap API. It automates data collection and offers visual insights into market trends.


### Technologies Used
- Python
- Jupyter Notebook
- Pandas
- Requests
- Seaborn
- Matplotlib

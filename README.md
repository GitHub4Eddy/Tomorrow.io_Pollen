# Tomorrow.io_Pollen
This QuickApp gives access to real-time pollen count and risk category for grass pollen, weed pollen and tree pollen of any location from Tomorrow.io

Pollen is a fine powder produced by trees and plants
Pollen can severely affect people, especially those with different ailments such as asthma and respiratory issues.

- Grass Pollen: Pollen grains from grasses
- Weed Pollen: Weeds are annual and perennial herbs and shrubs. A single plant may produce about a billion grains of pollen per season. 
- Tree Pollen: Pollen from trees such as Birch and Oak

Risk:
0: None
1: Very low
2: Low
3: Medium
4: High
5: Very High


IMPORTANT
- You need an API key from https://app.tomorrow.io
- The API is free up to 500 calls / per day, 25 calls / per hour, 3 calls / per second (Pollen is 5% of rate limit)
- You need to create your location in https://app.tomorrow.io/locations to get a location ID

ToDo:
- Automatic retrieval of location ID from Tomorrow.io
- Further testing of measurements


Version 0.1 (15th August 2021)
- Initial working version


Variables (mandatory): 
- apiKey = Get your free API key from Tomorrow.io
- location =  Tomorrow.io location ID (created in https://app.tomorrow.io/locations)
- interval = [number] in seconds time to get the data from the API
- timeout = [number] in seconds for http timeout
- debugLevel = Number (1=some, 2=few, 3=all, 4=simulation mode) (default = 1)
- icon = [numbber] User defined icon number (add the icon via an other device and lookup the number)

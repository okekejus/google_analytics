# Google Analytics Extraction 

My current workplace makes use of Google Analytics to capture website metrics. Many of these metrics are to be reported to a ministry at various intervals (monthly, quarterly, annually). Prior to my arrival, the retrieval of these metrics was done manually. 

## Business Need 
- The ability to retain static GA data in our data warehouses (GA data is often audited and changed by Google)
- Storage of data beyond the 14 month API limit
- Fetch page views, total users, new users, and user engagement for each web page AND the website overall
- Identify a subset of the web pages + tag them based on pre-identified criteria determined by a business unit

## Deployment 
This notebook is included in an Azure Synapse pipeline, which is set to run daily at 1AM. 

## Retrieval Logic 
Google Analytics is interesting in the way it returns requests for data. If I visit a site 3x in one week, I will be counted as a unique user for every day of the week, were I to request the data daily. If I were to request the data weekly, I would only be counted once. As a result, aggregating daily numbers from a GA report does not provide the correct total for a week, month, etc. There is also a 24 hour lag in reporting using the API method, which means I can only access data as recent as the previous day (11:59PM to be exact), at any given time. 

The above plays a huge role in why the script is set up the way it is. 

It takes the `current_date`, and creates a few variables using `datetime` and custom functions:
- previous_day: date
- fiscal_quarter: tuple (start_date, run_date)
- fiscal_year: tuple (start_date, run_date)
The functions account for the current day being equal to the last day of the fiscal year/quarter. If the current date (run_date) is not the end of the quarter/year, the current date is returned as the last day of the segment. 

Custom fiscal year functions have been created as our fiscal year is not the same as a calendar year. The above are then used to fetch the required metrics, using the following functions for the respective segments of time: 
- `fetch_metrics_aggregate(property_id, start_date, run_date)`
- `fetch_metrcis_per_page(property_id, start_date, run_date)`

## Output
The result of this script is 10 parquet files containing the requested metrics, per page and overall, for the time periods listed below:
- Daily (2 files, per page + overall)
- Weekly (2 files)
- Monthly (2 files)
- Quarterly (2 files)
- Annually (2 files)

All output files except Daily will be replaced each day with the most recent version. The weekly file will have the same name during the week, with Tuesday's version replacing Monday's version, Wednesday's replacing Tuesday's and so on, until the week changes and a new file is created based on the week start date. The same logic is applied to Monthly, Quarterly, and Annual outputs. 

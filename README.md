# myrepo_sg
CI repository

/* The below code creates a forecast model to predict the chicago crime for 2022 for a particular community area

#Create the Chicago Crime table from the Big Query dataset -> bigquery-public-data.chicago_crime.crime

create or replace table Chicago_Crime.Chicago_Crime_table as
(SELECT * FROM `bigquery-public-data.chicago_crime.crime`
where DATE(date) > '2020-01-01')

# Create the table Chicago Crime Table Model to be used for creating the model

create or replace table Chicago_Crime.Chicago_Crime_table_Model as

(SELECT
  primary_type,count(case_number) as total_cases,  date,  community_area
FROM
  `Chicago_Crime.Chicago_Crime_table`
  
  group by 
  (primary_type),
  (case_number),
  date,  community_area)
  order by primary_type,date
  

# Create the forecast model 
CREATE OR REPLACE MODEL
   `Chicago_Crime.Chicago_Crime_forecast_arima` 
    OPTIONS(MODEL_TYPE='ARIMA_PLUS',
    time_series_timestamp_col='date',
    time_series_data_col='total_cases',
    time_series_id_col=['primary_type','community_area']) 
AS
SELECT
  primary_type,total_cases,  date,  community_area
FROM
  `Chicago_Crime.Chicago_Crime_table_Model`

# Create the forecast data from the model

create or replace table Chicago_Crime.Chicago_Crime_forecast_data_180 AS

SELECT
      *
    FROM 
      ML.FORECAST(MODEL Chicago_Crime.Chicago_Crime_forecast_arima,
                  STRUCT(180 AS horizon, 
                         .9 AS confidence_level)
                 )

#Python code for text entry and output display
  
import tkinter as tk
import pandas as pd
from sqlalchemy import create_engine
from sqlalchemy.sql import text

engine = create_engine('sqlite://', echo=False)

forecast_data = pd.read_csv('C:\SG Backup _ 10_03_2020\sample.csv',header=None)

forecast_data.columns = ['violation','community_area','date','occurances']

forecast_data.to_sql('forecast_table',con=engine)

root= tk.Tk()

canvas1 = tk.Canvas(root, width = 800, height = 600,  relief = 'raised')
canvas1.pack()

label1 = tk.Label(root, text='Show the theft occurances for community area - 2 for 1/1/2022')
label1.config(font=('helvetica', 14))
canvas1.create_window(200, 25, window=label1)

label2 = tk.Label(root, text='Type Theft for the predictions:')
label2.config(font=('helvetica', 10))
canvas1.create_window(200, 100, window=label2)

entry1 = tk.Entry (root) 
canvas1.create_window(200, 140, window=entry1)


def getSquareRoot ():
    
    x1 = entry1.get()
       
        
    label3 = tk.Label(root, text= 'The community area 2 has ' + x1 + ' occurance for 1/1/2022:',font=('helvetica', 10))
    canvas1.create_window(200, 210, window=label3)
    
    sql_query = sqlalchemy.text("select violation,date, community_area from forecast_table where date = '1/1/2022' and violation='THEFT' and community_area ='2' ")
    result = connection.execute(sql_query)
    result_as_list = result.fetchall()

    for row in result_as_list:
        print(row)
    
    label4 = tk.Label(root, text=result_as_list,font=('helvetica', 10, 'bold'))
    canvas1.create_window(200, 230, window=label4)
    
button1 = tk.Button(text='Show the predictions', command=getSquareRoot, bg='brown', fg='white', font=('helvetica', 9, 'bold'))
canvas1.create_window(200, 180, window=button1)

root.mainloop()


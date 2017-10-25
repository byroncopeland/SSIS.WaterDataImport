# WaterML-Import
SSIS to retrieve JSON data from USGS and store in database

Pulls data from USGS Instantaneous Values Web service
Currently hardcoded for Tarrant County Tx
Package variables allow for select number of days back from current day to retrieve

Script Task creates web request and stores results to local .json file
SQL Task Process JSON parses file using SQL stored procedure Water.dbo.importJSON 
Data Flow inserts into final production table

Note: project originally designed to use WaterML data source, but later changed to JSON format

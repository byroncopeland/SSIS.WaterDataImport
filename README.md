# WaterML-Import
SSIS to retrieve JSON data from USGS and store in database

Pulls data from USGS Instantaneous Values Web service.
Currently hardcoded for Tarrant County Tx.

Package variables allow for select number of days back from current day to retrieve.

Script Task creates web request and stores results to local .json file
SQL Task Process JSON parses file using SQL stored procedure Water.dbo.importJSON 
Data Flow inserts into final production table

Note: project originally designed to use WaterML data source, but later changed to JSON format

Key coding
C# script

                string daysback = (string)Dts.Variables["User::DaysBack"].Value.ToString();
                string requeststring = "https://waterservices.usgs.gov/nwis/iv/?format=json&indent=on&countyCd=48439&period=P" + daysback + "D&parameterCd=00060,00065&siteType=ST&siteStatus=all";
                HttpWebRequest httpWebRequest = (HttpWebRequest)WebRequest.Create(requeststring);
                httpWebRequest.Method = WebRequestMethods.Http.Get;
                httpWebRequest.Accept = "application/json; charset=utf-8";
                 
                string file;
                var response = (HttpWebResponse)httpWebRequest.GetResponse();
                using (var sr = new StreamReader(response.GetResponseStream()))
                {
                    file = sr.ReadToEnd();
                }

                File.WriteAllText("c:\\water\\waterinstant.json", file);
                


importJSON Stored Procedure from water database (SQL 2016)

DECLARE @sourceinfo varchar(Max)
 SELECT @sourceinfo = Bulkcolumn 
   FROM OPENROWSET (BULK 'c:\water\waterinstant.json', SINGLE_BLOB) as j
 
 INSERT dbo.instantstage
 select JSON_Value (c.value, '$.name') as name
		,JSON_Value (c.value, '$.sourceInfo.siteName') as site
		,JSON_Value (p.value, '$.value') as parameter
		,JSON_Value (vv.value, '$.value') as value
		,JSON_Value (vv.value, '$.dateTime') as dtmeasure
 from OPENJSON(@sourceinfo, '$.value.timeSeries') as C
 CROSS APPLY OPENJSON(c.value, '$.variable.variableCode') as P 
 CROSS APPLY OPENJSON(c.value, '$.values') as V
 CROSS APPLY OPENJSON(v.value, '$.value') as vv
 

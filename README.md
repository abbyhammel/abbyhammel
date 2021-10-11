#Importing the CSV flies into the server:
scp /Users/abbyhammel/Documents/NationalParks.csv abbyhammel@139.147.9.185:/home/abbyhammel

#Login to server:
ssh abbyhammel@139.147.9.185

#Change code to sql:
psql

#Create table from national park csv:
create table nationalpark (
park_code varchar,
park_name varchar,
state varchar,
acres float,
latitude float,
longitude float);

\COPY nationalpark FROM  /home/abbyhammel/NationalParks.csv DELIMITER ',' CSV HEADER;

#Importing the CSV flies into the server:
scp /Users/abbyhammel/Documents/flora_fauna.csv abbyhammel@139.147.9.185:/home/abbyhammel

#Login to server:
ssh abbyhammel@139.147.9.185

#Change code to sql:
psql

#Create table from flora fauna csv:
create table flora_fauna
(species_id varchar,
park_name varchar,
category varchar,
orders varchar,
family varchar,
scientific_name varchar,
common_names varchar,
record_status varchar,
occurrence varchar,
nativeness varchar,
abundance varchar,
seasonality varchar,
conservation_status varchar,
endangerment varchar)

\COPY flora_fauna FROM  /home/abbyhammel/flora_fauna.csv DELIMITER ',' CSV HEADER;

#Create joint table of both csv's, joined by park name:
create table acombo as
select * from flora_fauna 
 join nationalpark using (park_name);
 
 #Filter the data with sql
 
#Creates a table with park names and latitudes of animals that are endangered
create table pt2 as select park_name,latitude from acombo where "conservation_status"='Endangered'

#creates a table with the national park names and counts of how many endangered species there are within them
Create table namecount as
SELECT park_name, COUNT(*) 
FROM pt2 
GROUP BY park_name order by 2;

#creates table with park name, latitude and counts of endangered species y joining the previous two tables
create table lat as
select * from pt2 
 join namecount using (park_name);
 
#creates table with park name, latitude and counts of endangered species with no repeats
create table latitudes as select distinct park_name, latitude, count from lat order by count;

#Creates a table with park names and longitudes of animals that are endangered
create table pt3 as select park_name,longitude from acombo where "conservation_status"='Endangered';

#creates table with park name, latitude and counts of endangered species y joining the previous two tables
create table long as
select * from pt3 
 join namecount using (park_name);

#creates table with park name, latitude and counts of endangered species with no repeats
create table longitudes as select distinct park_name, longitude, count from long order by count;

#Creating a script and csv files
#create python scripts, I made three for my three tables
  touch pythlab_script.py
  touch longitude.py
  touch latititude.py
  
 #edit python script
  vi pythlab_script.py
  vi longitude.py
  vi latititude.py
  
#creates CSV file from tables created. This is one example, but I changed the csv name and query to save others. 
import psycopg2
import csv
import os
from csv import reader


connection = psycopg2.connect(user = "abbyhammel",
                              password = "201",
                              host = "localhost",
                              port = "5432",
                              database = "abbyhammel")

cursor = connection.cursor()
create_table_query = "select * from acombo;"
cursor.execute( create_table_query )
rows = cursor.fetchall()
    
headers = [i[0] for i in cursor.description]

csvFile = csv.writer(open('acombo.csv', 'w'),delimiter=',')

csvFile.writerow(headers)
for row in rows:
	csvFile.writerow( row ) 
connection.commit()
print("table created")

except (Exception, psycopg2.Error) as error :
print ("Error while connecting to PostgreSQL", error)


if(connection):
	cursor.close()
	connection.close()
	print("PostgreSQL connection is closed")
  
  
#create R script
touch Rlab2_script.R

#edit R script
vi Rlab2_script.R
#In script to show summaery statisticss
if (!require("tidyverse")) install.packages("tidyverse", dependencies=T)
library(tidyverse)

longitude <- read.csv("longitude.csv")

latitude <- read.csv("latitude.csv")

summary(longitude)
summary(latitude)

ggplot(data = latitude) +
   geom_point(mapping = aes(x = latitude, y = count))








# Project Description
This project aims to bring together data regarding city Real Estate and Demographics. The data is pulled from three separate datasets:
   
   [Home Values](https://public.opendatasoft.com/explore/dataset/unites-states-home-values/table/)
   
   [Rental Values](https://public.opendatasoft.com/explore/dataset/rental-values/table/?disjunctive.regionname&disjunctive.city&disjunctive.state&disjunctive.metro&disjunctive.countyname)
   
   [City Demographics](https://public.opendatasoft.com/explore/dataset/us-cities-demographics/table/)

## The idea is to create high grade data pipelines that:
    - are dynamic
    - built from reusable tasks
    - can be monitored
    - allow easy backfills
  
 ## Database schema design

**Staging Tables**
 - Home_Values
 - Rental_Values
 - Demographics

**Fact Table**
  - City_Housing_Demographics - records information about home and rental values and demographics information on a city level.
      - *City, State, Home_Value, Rental_Value, Race, Median_Age, Male_Population, Female_Population, Total_Population*

**Dimension Table**
  - City_Housing_Costs - details housig costs per city
      - *City, State, Home_Value,Rental_Type, Rental_Value, Avg_Household_Size*
  


# Project structure
This project includes the following script files:
  - **Dag Files**
    - *dags/main_dags.py*: Directed Acyclic Graph (DAG) definition with imports, tasks and task dependencies.
    - *dags/sub_dags.py*: Subdag designed to handle loading of Dimensional table tasks.
  - **helper Class**
    - *plugins/helpers/sql_queries.py*: Contains Insert SQL statements.
  - **SQL Tables** 
    - *plugins/operators/create_tables.sql*: Contains SQL Table creations statements.
  - **Operators**
    - *plugins/operators/stage_redshift.py*: Operator copies data from S3 buckets into redshift staging tables.
    - *plugins/operators/load_dimension.py*: Operator loads data from redshift staging tables into dimensional tables.
    - *plugins/operators/load_fact.py*: Operator loads data from redshift staging tables into fact table.
    - *plugins/operators/data_quality.py*: Operator validates data quality in redshift tables.

# DAG Configuration Parameters:

    - The DAG does not have dependencies on past runs
    - DAG has schedule interval set to hourly
    - On failure, the tasks are retried 3 times
    - Retries happen every 5 minutes
    - Catchup is turned off
    - Do not email on retry
  

 # Project DAG Operators
We built four different operators that will stage the data, transform the data, and run checks on data quality. All of the operators and task instances run SQL statements against the Redshift database. Additionally, using parameters allows us to build flexible, reusable, and configurable operators that we can later apply to many kinds of data pipelines with Redshift and with other databases.

  **Stage Operator**
    
  The stage operator can load any CSV formatted files from S3 to Amazon Redshift. it creates and runs a SQL COPY statement based on the parameters provided. The operator's parameters specify where in S3 the file is loaded and what is the target table.

 
  **Fact and Dimension Operators**
    
  SQL helper class helps us run data transformations. Most of the logic is within the SQL transformations and the operator takes as input a SQL statement and target database on which to run the query against. The target table that will contain the results of the transformation.

  Dimension loads are done with the truncate-insert pattern where the target table is emptied before the load. Thus, we have a parameter that allows switching between insert modes when loading dimensions. (Fact tables are usually so massive that they should only allow append type functionality.)

  **Data Quality Operator**
    
  The final operator is the data quality operator, which is used to run checks on the data itself. The operator's main functionality is to receive one or more SQL based test cases along with the expected results and execute the tests. For each test, the test result and expected result needs to be checked and if there is no match, the operator raises an exception and the task retries and fails eventually.

  *For example*: one test could be a SQL statement that checks if certain column contains NULL values by counting all the rows that have NULL in the column. We do not want to have any NULLs so expected result would be 0 and the test would compare the SQL statement's outcome to the expected result.
  
  # Future Scenarios
  
  Given that this is run utlizing the power of Airflow and Redshift, we can leverage the tools' capabilities to address a variety of situations.
  
  If our data were to increase a hundred fold, we would consider altering the folder structure so that it can be incrementally loaded at the appropriate time. 
  for example, we can divide out files based on date. 
 
 
 Luckily, using Airflow we can schedule our jobs and run them nightly, so that our data would be ready every morning.
 
 lastly, If our data ever needed to be accessed by more than 100 people, we could leverage Redshift's horizontal scalability features by inreasing the number of available nodes. 

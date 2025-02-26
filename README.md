Here is your **README.md** file for the GitHub repository:  


# Web Server Log Analysis using Apache Hive

## Project Overview
This project analyzes web server logs using **Apache Hive**. The dataset consists of web requests stored in a **CSV format**, and we use **HiveQL** to extract insights about website traffic patterns. The key objectives of this project are:
- Counting the total number of web requests.
- Analyzing HTTP status codes.
- Identifying the most visited pages.
- Analyzing the most common user agents (browsers).
- Detecting suspicious activity based on failed requests.
- Observing traffic trends over time.
- Implementing partitioning to optimize queries.

---

## Approach and Implementation
### **Data Processing in Hive**
1. **Created an External Hive Table**  
   - The table reads data from **HDFS** without modifying its storage.
   - Data fields are separated by commas (`FIELDS TERMINATED BY ','`).
  
2. **Data Loading and Query Execution**  
   - The CSV file was uploaded to HDFS and linked to Hive.
   - Queries were executed to extract insights.

3. **Partitioning for Optimization**  
   - A **partitioned table** was created to store data based on **status code**.
   - This improves query performance when filtering by status.

---

## Execution Steps

### **1. Setup Docker and Copy Data to HDFS**
```bash
docker cp web_server_logs.csv namenode:/tmp/
docker exec -it namenode /bin/bash
hdfs dfs -mkdir -p /user/hive/warehouse/web_logs
hdfs dfs -put /tmp/web_server_logs.csv /user/hive/warehouse/web_logs/
```

### **2. Create Hive Database and External Table**
```sql
CREATE DATABASE web_logs;
USE web_logs;

CREATE EXTERNAL TABLE web_server_logs (
    ip STRING,
    timestamp_ STRING,
    url STRING,
    status INT,
    user_agent STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hive/warehouse/web_logs/';
```

### **3. Load Data into Hive**
```sql
LOAD DATA INPATH '/user/hive/warehouse/web_logs/web_server_logs.csv' INTO TABLE web_server_logs;
```

### **4. Execute Queries**
#### **Total Web Requests**
```sql
SELECT COUNT(*) AS total_requests FROM web_server_logs;
```

#### **Analyze HTTP Status Codes**
```sql
SELECT status, COUNT(*) AS count FROM web_server_logs GROUP BY status;
```

#### **Identify Most Visited Pages**
```sql
SELECT url, COUNT(*) AS visits FROM web_server_logs GROUP BY url ORDER BY visits DESC LIMIT 3;
```

#### **Analyze Most Common User Agents**
```sql
SELECT user_agent, COUNT(*) AS count FROM web_server_logs GROUP BY user_agent ORDER BY count DESC;
```

#### **Detect Suspicious Activity (IP Addresses with >3 Failed Requests)**
```sql
SELECT ip, COUNT(*) AS failed_requests 
FROM web_server_logs 
WHERE status IN (404, 500) 
GROUP BY ip 
HAVING COUNT(*) > 3;
```

#### **Analyze Traffic Trends (Requests Per Minute)**
```sql
SELECT SUBSTR(timestamp_, 1, 16) AS time_slot, COUNT(*) AS request_count 
FROM web_server_logs 
GROUP BY SUBSTR(timestamp_, 1, 16) 
ORDER BY time_slot;
```

---

## Challenges Faced & Solutions
| **Challenge** | **Solution** |
|--------------|-------------|
| Hive did not recognize the `timestamp` column | Used `timestamp_` as the correct column name. |
| Dynamic partitioning failed due to strict mode | Enabled **nonstrict** mode using `SET hive.exec.dynamic.partition.mode = nonstrict;`. |
| Hive query failed due to missing `/user/hive/warehouse/web_logs/` | Created the directory manually using `hdfs dfs -mkdir -p`. |
| Exporting results from HDFS to local machine | Used `hdfs dfs -get` and `docker cp` to transfer files. |

---

## Sample Input and Output

### **Sample Input (web_server_logs.csv)**
```
ip,timestamp_,url,status,user_agent
192.168.1.1,2024-02-01 10:15:00,/home,200,Mozilla/5.0
192.168.1.2,2024-02-01 10:16:00,/products,200,Chrome/90.0
192.168.1.3,2024-02-01 10:17:00,/checkout,404,Safari/13.1
192.168.1.10,2024-02-01 10:18:00,/home,500,Mozilla/5.0
192.168.1.15,2024-02-01 10:19:00,/products,404,Chrome/90.0
```

### **Sample Output**
#### **Total Web Requests**
```
Total Requests: 100
```
#### **Status Code Analysis**
```
200: 80
404: 10
500: 10
```
#### **Most Visited Pages**
```
/home: 50
/products: 30
/checkout: 20
```
#### **Traffic Source Analysis**
```
Mozilla/5.0: 60
Chrome/90.0: 30
Safari/13.1: 10
```
#### **Suspicious IP Addresses**
```
192.168.1.10: 5 failed requests
192.168.1.15: 4 failed requests
```
#### **Traffic Trend Over Time**
```
2024-02-01 10:15: 5 requests
2024-02-01 10:16: 7 requests
```

---

## **How to Retrieve Results**
After running the queries, results are stored in HDFS. You can retrieve them using:
```bash
hdfs dfs -cat /user/hive/output/web_logs_analysis/*
```
To copy results to your local machine:
```bash
docker exec -it namenode hdfs dfs -get /user/hive/output/web_logs_analysis /tmp/
docker cp namenode:/tmp/web_logs_analysis ./web_logs_analysis_results
ls -l ./web_logs_analysis_results
```

---

## Conclusion
This project demonstrates how to use **Apache Hive** to process **web server logs**, extract insights, and optimize queries using partitioning. 

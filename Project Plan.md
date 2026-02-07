# **Project Plan: Enterprise Log File Analysis (Chargebee Case Study)**

**Objective:** Build a robust Google Colab notebook to analyze one year of server log data to diagnose crawl budget waste, identify spider traps, and explain the "Crawled \- Currently Not Indexed" phenomenon.

**Target Environment:** Google Colab

**Language:** Python

**Libraries:** Pandas, Matplotlib, Seaborn, Regex (re), Glob, Gspread (for bonus)

## **Phase 1: Environment & Dependency Setup**

**Instruction to Agent:**

1. **Mount Google Drive:** Create a cell to mount Google Drive to access the raw log files.  
2. **Import Libraries:** Import necessary libraries.  
   * pandas (Data manipulation)  
   * numpy (Math)  
   * re (Regular Expressions for parsing)  
   * glob (File path pattern matching)  
   * matplotlib.pyplot & seaborn (Visualization)  
   * gzip (If logs are compressed)  
   * gspread & google.auth (For the bonus Google Sheets export)

\# Expected path structure to look for:  
\# /content/drive/MyDrive/SEO\_Project/Logs/  
\# /content/drive/MyDrive/SEO\_Project/Crawl\_Data/ (Optional, for gap analysis)

## **Phase 2: Data Ingestion (The Multi-File Logic)**

**Context:** The data spans a full year. We cannot simply load one file. The script must iterate through a directory, read multiple log files (likely .log, .txt, or .gz), parse them, and combine them into a single DataFrame.

**Instruction to Agent:**

1. **Define Regex Pattern:** Use the standard Combined Log Format pattern (Apache/Nginx).  
   * *Pattern:* r'^(\\S+) \\S+ \\S+ \\\[(\[\\w:/\]+\\s\[+\\-\]\\d{4})\\\] "(\\S+) (\\S+)\\s\*(\\S\*)" (\\d{3}) (\\S+) "(\[^"\]\*)" "(\[^"\]\*)"'  
   * *Fields:* IP, Timestamp, Method, Request URL, Protocol, Status Code, Bytes, Referrer, User Agent.  
2. **Batch Processing Function:** Create a function ingest\_logs(directory\_path) that:  
   * Uses glob to find all files in the folder.  
   * Iterates through each file.  
   * Reads line-by-line (to manage memory).  
   * Applies the Regex.  
   * Appends valid records to a list.  
   * Converts the final list to a Pandas DataFrame.  
3. **Optimization Note:** If the dataset is massive, instruct the agent to parse *only* lines containing "Googlebot" to save memory during the initial read.

## **Phase 3: Data Cleaning & Pre-processing**

**Instruction to Agent:**

1. **Timestamp Conversion:** Convert the timestamp column to datetime objects. Set this as the DataFrame index for easier time-series resampling.  
2. **User Agent Filtering:** Filter the DataFrame to ensure we are only analyzing Googlebot.  
   * *Filter:* String contains "Googlebot" (Case insensitive).  
   * *Note:* Add a comment that in a live production environment, we would perform rDNS verification here, but for this static project, we accept the UA string.  
3. **File Type Categorization:** Create a new column file\_type.  
   * Logic: Check the URL extension.  
   * Categories: HTML (no extension or .html), JS (.js), CSS (.css), Image (.jpg, .png, .gif, .svg), Data (.json, .xml).  
4. **URL Normalization:** Create a clean\_url column.  
   * Logic: Strip all query parameters (everything after ?) to analyze the core page vs. the parameterized version.

## **Phase 4: Core Analysis Modules**

### **Module A: Crawl Volume & Frequency (The Baseline)**

**Instruction to Agent:**

1. **Daily Hits:** Resample the data by Day ('D') and count requests. Plot a Line Chart.  
2. **Bot Breakdown:** If multiple Googlebot types exist (Smartphone vs. Desktop vs. Image), show the distribution.

### **Module B: Status Code Health (Efficiency)**

**Instruction to Agent:**

1. **Pivot Table:** Group by status\_code and file\_type.  
2. **304 vs 200 Check:** Calculate the percentage of 304 Not Modified responses for Static Resources (JS/CSS).  
   * *Insight Logic:* If JS/CSS are 99% 200 OK, flag this as a "Cache Control Opportunity."  
3. **Error Identification:** Filter for 5xx (Server Errors) and 4xx (Client Errors) to identify specific URLs causing failures.

### **Module C: Spider Trap Detection (The "Not Indexed" Culprit)**

**Instruction to Agent:**

1. **Cardinality Check:** Group by clean\_url (the base path) and count the number of *unique* request\_url (full URL with params).  
2. **Thresholding:** Identify base paths with \> 50 unique variations.  
   * *Example:* If /pricing exists as a base, but we see /pricing?currency=usd, /pricing?sort=desc, etc., calculate how many variations exist.  
3. **Output:** List the Top 10 directories consuming the most budget via parameter generation.

### **Module D: Rendering Budget Impact**

**Instruction to Agent:**

1. **Byte Volume:** Sum the bytes\_sent column grouped by file\_type.  
2. **Comparison:** Compare total bandwidth spent on HTML vs. Javascript/CSS.  
3. **Visualization:** Create a Pie Chart showing "Crawl Budget by Content Type" (HTML vs Resources).

## **Phase 5: Visualization & Reporting**

**Instruction to Agent:**

Generate the following visualizations using Matplotlib/Seaborn:

1. **Time Series:** Daily Googlebot Requests over the year.  
2. **Bar Chart:** Top 10 Most Crawled Directories.  
3. **Bar Chart:** Status Code Distribution (Color-coded: 200=Green, 30x=Yellow, 40x=Orange, 50x=Red).  
4. **Scatter Plot (Optional):** Response Time (if available in logs) vs. Time of Day.

## **Phase 6: The "Bonus" Integration (Google Sheets)**

**Instruction to Agent:**

1. **Auth Setup:** Include the google.colab.auth.authenticate\_user() code block.  
2. **Export Function:** Create a function to write the specific "Spider Trap" and "Status Code" summary DataFrames to a new Google Sheet named "Chargebee\_Audit\_Findings".  
3. **Output:** Print the URL of the generated Google Sheet.

## **Technical Considerations for the Agent**

* **Memory Management:** Since we are parsing a year of logs, avoid keeping the "raw" list in memory after creating the DataFrame. Use del on large variables and run gc.collect().  
* **Error Handling:** Include try/except blocks in the file reading loop to prevent one corrupt log file from crashing the entire ingestion process.
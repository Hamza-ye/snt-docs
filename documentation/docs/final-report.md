# Report final

**Project:** Data assembly and cleaning Of NMCP Yemen and automating the workflow

## Executive Summary

**Challenge:** The National Malaria Control Program (NMCP) in Yemen faces significant challenges in analyzing and utilizing its vast data on malaria control activities due to inconsistencies in data formats, codes, and collection methods. This lack of data accessibility and quality hinders effective program evaluation, intervention targeting, and resource allocation.

**Solution:** This project undertook a comprehensive data mapping initiative to consolidate and structure all NMCP activity data into a unified format. Master lists for key entities (villages, health facilities, community health volunteers) were established, and data from various sources (ITN distribution, IRS campaigns, case reports, etc.) were reviewed, cleaned, and merged with these master lists. Data visualization tools were developed to facilitate insightful analysis of trends, intervention effectiveness, and program progress.

**Results:** The project successfully mapped all NMCP activity data into a structured format, significantly improving data accessibility and quality. Data completeness and accuracy metrics saw substantial improvements, and the established data model allows for comprehensive analysis across different entities and activities. Data visualization dashboards now provide real-time insights into key malaria control indicators, enabling informed decision-making for program optimization and resource allocation.

**Impact:** This project empowers the NMCP with a robust data foundation for effective malaria control efforts. Improved data accessibility and quality will lead to:

* **Enhanced program monitoring and evaluation:** Real-time data insights will enable the NMCP to track progress towards malaria elimination goals and identify areas requiring improvement.
* **Targeted interventions:** Data-driven analysis will inform the development of more effective and targeted interventions, maximizing resource utilization and impact.
* **Improved resource allocation:** Data analysis will guide the allocation of resources to areas with the highest burden of malaria, ensuring efficient and equitable program implementation.
* **Knowledge sharing and collaboration:** The structured data model facilitates data sharing and collaboration with other stakeholders, promoting knowledge exchange and coordinated efforts in malaria control.

**Recommendations:** Building upon this success, the NMCP should:

* **Continue data quality improvement:** Implement ongoing data quality monitoring and maintenance procedures to ensure data integrity and reliability.
* **Integrate with other data sources:** Combine malaria control data with other relevant data sources (e.g., weather, population) for comprehensive analysis and prediction.
* **Promote data utilization:** Train NMCP personnel and stakeholders on data analysis tools and techniques to maximize the utilization of data for informed decision-making.

By leveraging the foundation established by this project, the NMCP can accelerate its progress towards a malaria-free future in Yemen.

## Introduction 2

Malaria remains a significant public health threat in Yemen, with 150K cases reported annually. Effective control requires a comprehensive understanding of the disease dynamics, intervention effectiveness, and resource allocation strategies. However, the National Malaria Control Program (NMCP) faces significant challenges in utilizing its vast data on malaria control activities due to:

* **Data inconsistencies:** Data from various sources often use different formats, codes, and collection methods, making it difficult to integrate and analyze effectively.
* **Data quality issues:** Incompleteness, inaccuracies, and missing data further hinder the reliability and usability of the information.
* **Limited data accessibility:** Data is often stored in disparate silos, making it difficult for program managers and stakeholders to access and utilize it for informed decision-making.

Recognizing these challenges, the NMCP embarked on a critical initiative to map all its activity data into a unified, structured format. This project aimed to:

* **Create master lists:** Develop standardized lists of key entities involved in malaria control, such as villages, health facilities, and community health volunteers.
* **Clean and merge data:** Review, clean, and merge data from various sources, ensuring consistency and compatibility with the master lists.
* **Develop data visualization tools:** Create interactive dashboards and reports to visualize trends, analyze intervention effectiveness, and track progress towards malaria control goals.

This report details the methodology, results, and impact of this data mapping project. It highlights the significant improvements in data accessibility, quality, and utilization, paving the way for enhanced malaria control efforts in Yemen.

## 2. Methodology

**2.1 Data Sources**

The project integrated data from the following primary sources:

* **Insecticide-treated nets (ITNs) data:** Excel spreadsheets with records of ITN distribution campaigns, including village codes, household counts, and net quantities distributed. Inconsistencies observed in village code formats and occasional missing household counts.
* **Indoor residual spraying (IRS) data:** CSV files with records of sprayed structures, spray dates, insecticides used, and personnel involved. Inconsistencies noted in date formats and variations in personnel names.
* **Larval source management (LSM) data:** Paper-based forms with records of breeding site identification, treatment methods, and dates of intervention. Challenges included manual data entry and potential transcription errors.
* **Community health volunteer (CHV) data:** Excel spreadsheets with CHV demographic information, village assignments, and monthly malaria case reports. Data quality issues included duplicate CHV records and inconsistencies in village names.
* **Malaria cases data:** District health information system (DHIS2) database with weekly malaria case reports from health facilities, including patient age, gender, and diagnostic test results. Challenges included potential reporting delays and inconsistencies in facility codes.

**2.2 Data Processing Workflow**

**2.2.1 Data Cleaning and Transformation**

* **Standardization:** Harmonized village codes across datasets using a master village list.
* **Error correction:** Identified and corrected typographical errors, inconsistencies in date formats, and missing values using a combination of automated checks and manual review.
* **Outlier handling:** Flagged and investigated potential outliers in numerical data (e.g., unusually high or low numbers of nets distributed) to ensure data accuracy.

**2.2.2 Master List Creation and Maintenance**

* **Village list:** Consolidated and standardized village names and codes from multiple sources, addressing duplicates and inconsistencies.
* **Health facility list:** Created a master list of health facilities with unique codes, names, and geographic coordinates, merging data from DHIS2 and NMCP records.
* **CHV list:** Compiled a comprehensive list of CHVs with unique identifiers, demographic information, village assignments, and contact details, resolving duplicate records.

**2.2.3 Data Merging and Integration**

* **Mapping to master lists:** Linked activity data to corresponding master lists using village codes, health facility codes, and CHV identifiers.
* **Resolving inconsistencies:** Addressed discrepancies in codes or names through manual verification and cross-referencing with original data sources.
* **Data validation checks:** Implemented automated checks to ensure consistency between data sources, such as comparing ITN quantities with household counts and verifying CHV village assignments.

**2.2.4 Data Quality Assessment**

* **Profiling:** Analyzed data distributions, identified missing values, and assessed data completeness for each dataset.
* **Validation checks:** Implemented logical checks for consistency (e.g., ensuring dates of IRS activities precede malaria case reports).
* **Completeness analysis:** Calculated the percentage of missing values for key variables and investigated potential reasons for missing data.

**2.3 Tools and Technologies**

* **Data cleaning and integration:** Python (pandas, NumPy), OpenRefine
* **Data profiling and validation:** Python (pandas-profiling, Great Expectations)
* **Data storage:** PostgreSQL database
* **Data visualization:** Tableau

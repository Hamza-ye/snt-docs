# Report final

**Project:** Data assembly and cleaning Of NMCP Yemen and automating the workflow

## Executive Summary

**Challenge:** The National Malaria Control Program (NMCP) in Yemen faces significant challenges in analyzing and utilizing its vast data on malaria control activities due to inconsistencies in data formats, codes, and collection methods. This lack of data accessibility and quality hinders effective program evaluation, intervention targeting, and resource allocation.

**Approach:** The project used a Collaborative Approach Engaging a cross-functional team to collaboratively address inconsistencies in data formats, Actively involving *Risk analysis team* in harmonization decisions, ensuring alignment with program goals, and Iteratively refining entity relationships based on feedback from them and NMCP management. The collaborative approach has resulted in a coherent dataset that reflects the collective expertise of the team.

**Result:** This project successfully mapped all NMCP activity data into a structured format, significantly improving data accessibility and quality. Data completeness and accuracy metrics saw substantial improvements, and the established data model allows for comprehensive analysis across different entities and activities. Data visualization dashboards now provide real-time insights into key malaria control indicators, enabling informed decision-making for program optimization and resource allocation. Any Future data can be easily checked, validated, cleaned, and merged using the automated solution and the pipeline developed in this project.

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

## Introduction

Malaria remains a significant public health threat in Yemen, with 150K cases reported annually. Effective control requires a comprehensive understanding of the disease dynamics, intervention effectiveness, and resource allocation strategies. However, the National Malaria Control Program (NMCP) faces significant challenges in utilizing its vast data on malaria control activities due to:

* **Data inconsistencies:** Data from various sources often use different formats, codes, and collection methods, making it difficult to integrate and analyze effectively.
* **Data quality issues:** Incompleteness, inaccuracies, and missing data further hinder the reliability and usability of the information.
* **Limited data accessibility:** Data is often stored in disparate silos, making it difficult for program managers and stakeholders to access and utilize it for informed decision-making.

Recognizing these challenges, the NMCP embarked on a critical initiative to map all its activity data into a unified, structured format. This project aimed to:

* **Create master lists:** Review and Update the standardized lists available of key entities involved in malaria control, such as villages, health facilities, and community health volunteers. and develop the missing ones needed for the mapping process.
* **Connect/Define Catchment of each health facilities:** connect health facilities with their corresponding catchment villages. Building on an Existing mapping and utilizing a data-driven approach and robust methodologies, to establish a detailed catchment map that empowers informed resource allocation and optimizes service delivery within the region.
* **Automated Data Pipelines (Pentaho and Python-Powered):** Implement lately to streamlined data processing and maintained data integrity throughout the analysis by employing Pentaho to orchestrate data extraction, and transformation processes. create scripts to address some data inconsistencies, including standardization, and normalization.
* **Clean and merge data:** Review, clean, and merge data routine/not routine data from various sources, ensuring consistency and compatibility with the master lists.
* **Develop data visualization tools:** Create interactive dashboards and reports to visualize trends, analyze intervention effectiveness, and track progress towards malaria control goals.
* **Implement testing:** to guarantee data integrity and consistency before merging into the production Database.

This report details the methodology, results, and impact of this data mapping project. It highlights the significant improvements in data accessibility, quality, and utilization, paving the way for enhanced malaria control efforts in Yemen.

## 2. Methodology

### 2.1 Data Sources

The project integrated data from the following primary sources:

#### 2.1.1 Insecticide-treated nets (ITNs) Data

Excel spreadsheets with records of ITN distribution campaigns, including village codes, household counts, and net quantities distributed. Inconsistencies observed in village code formats and occasional missing household countsو Lack of essential information like village names, team IDs. Incorrect spellings of village names of villages without code. Mismatches in submission_time formats. Changing in Form structure from 2023 onward

**format:**

```

| Name                     | Type       | describtion                    |
|------------------------- |----------- |------------------------------- |
| report_target_type       | int        | is it within target or not     |
| report_ppc_code          | int        | location code                  |
| report_team_no           | int        | team no reached this location  |
| day_reached              | varchar    | day this location reached      |
| report_houses            | int        |                                |
| report_residents         | int        |                                |
| report_idps              | int        |                                |
| report_idps_individuals  | int        |                                |
| report_population        | int        |                                |
| report_males             | int        |                                |
| report_females           | int        |                                |
| report_male_children     | int        |                                |
| report_female_children   | int        |                                |
| report_pregnants         | int        |                                |
| report_itns_distributed  | int        |                                |
| report_submission_time   | timestamp  |                                |
| report_is_idps_camp      | bool       | is this location an idps_camp  |
| report_tlcommenet        | text       | team leader comments           |

```

* **Impact of the inconsistencies in ITNs data:** the number of records affected or the degree of variation in data values.

#### 2.1.2 Indoor Residual Spraying (IRS) Data

CSV files with records of sprayed structures, spray dates, insecticides used, and personnel involved. Inconsistencies noted in date formats and variations in personnel names.

* **Inconsistencies and Issues in IRS data:** Inconsistencies in itns data includes Lack of essential information like village names, team IDs. Incorrect spellings of village names, team leader names. Mismatches in date of submission_time. Changing in Form structure from 2023 onward.

* **Impact of the inconsistencies in IRS data:** the number of records affected or the degree of variation in data values.

#### 2.1.3 Larval Source Management (LSM) Data

Paper-based forms with records of breeding site identification, treatment methods, and dates of intervention. Challenges included manual data entry and potential transcription errors.

* **Inconsistencies and Issues in LSM data:** Inconsistencies in itns data includes Lack of essential information like village names, team IDs. Incorrect spellings of village names, team leader names. Mismatches in date of submission_time. Changing in Form structure from 2023 onward.

* **Impact of the inconsistencies in LSM data:** the number of records affected or the degree of variation in data values.

#### 2.1.4 Community Health Volunteer (CHV) Data

Excel spreadsheets with CHV demographic information, village assignments, and monthly malaria case reports. Data quality issues included duplicate CHV records and inconsistencies in village names.

* **Inconsistencies and Issues in CHV data:** Inconsistencies in itns data includes Lack of essential information like village names, team IDs. Incorrect spellings of village names, team leader names. Mismatches in date of submission_time. Changing in Form structure from 2023 onward.

* **Impact of the inconsistencies in CHV data:** the number of records affected or the degree of variation in data values.

#### 2.1.5 Malaria Cases Data

District health information system (DHIS2) database with weekly malaria case reports from health facilities, including patient age, gender, and diagnostic test results. Challenges included potential reporting delays and inconsistencies in facility codes.

* **Inconsistencies and Issues in Malaria cases data:** Inconsistencies in itns data includes Lack of essential information like village names, team IDs. Incorrect spellings of village names, team leader names. Mismatches in date of submission_time. Changing in Form structure from 2023 onward.

* **Impact of the inconsistencies in Malaria cases data:** the number of records affected or the degree of variation in data values.

#### 2.1.6 Administrative Data

Were initially developed for administrative use, not for public health surveillance and have a larger coverage of population and Areas. For example, GIS (Geographical Information System/GPS/Geodata), Health Facilities information, Community Health Volunteers list, etc. connecting health facilities with their corresponding catchment villages.

* **Inconsistencies and Issues in Administrative data:** Inconsistencies in itns data includes Lack of essential information like village names, team IDs. Incorrect spellings of village names, team leader names. Mismatches in date of submission_time. Changing in Form structure from 2023 onward.

* **Impact of the inconsistencies in Administrative data:** the number of records affected or the degree of variation in data values.

### 2.2 Processing Workflow

**2.2.1 Data Cleaning and Transformation:**

* **Standardization:** Harmonized village codes across datasets using a master village list.
* **Error correction:** Identified and corrected typographical errors, inconsistencies in date formats, and missing values using a combination of automated checks and manual review.
* **Outlier handling:** Flagged and investigated potential outliers in numerical data (e.g., unusually high or low numbers of nets distributed) to ensure data accuracy.

**2.2.2 Master List Creation and Maintenance:**

* **Village list:** Consolidated and standardized village names and codes from multiple sources, addressing duplicates and inconsistencies.
* **Health facility list (HF):** Created a master list of health facilities with unique codes, names, and geographic coordinates, merging HFs from EIdews Malaria cases reporting system and NMCP records.

![Organization Unit](images/orgunits.png){ align=center, width="400" }

![Merged Hfs](images/merged-hfs.png){ align=center, width="450" }

* **CHV list:** Compiled a comprehensive list of CHVs with unique identifiers, demographic information, village assignments, and contact details, resolving duplicate records.

#### 2.2.3 Data Merging and Integration

* **Mapping to master lists:** Linked activity data to corresponding master lists using village codes, health facility codes, and CHV identifiers.
* **Resolving inconsistencies:** Addressed discrepancies in codes or names through manual verification and cross-referencing with original data sources.
* **Data validation checks:** Implemented automated checks to ensure consistency between data sources, such as comparing ITN quantities with household counts and verifying CHV village assignments.

#### 2.2.4 Data Quality Assessment

* **Profiling:** Analyzed data distributions, identified missing values, and assessed data completeness for each dataset.
* **Validation checks:** Implemented logical checks for consistency (e.g., ensuring dates of IRS activities precede malaria case reports).
* **Completeness analysis:** Calculated the percentage of missing values for key variables and investigated potential reasons for missing data.

### 2.3 Tools and Technologies

* **Data cleaning and integration:** Python (pandas, NumPy), OpenRefine
* **Data profiling and validation:** Python (pandas-profiling, Great Expectations)
* **Data storage:** PostgreSQL database
* **Data visualization:** Tableau

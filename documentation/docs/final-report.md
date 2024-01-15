# Report final

**Project:** Data assembly and cleaning Of NMCP Yemen and automating the workflow

## Executive Summary

The National Malaria Control Program (NMCP) in Yemen faced a critical challenge: fragmented and inconsistent data across its various malaria control activities. This hampered program effectiveness, hindering the ability to analyze interventions, target resources, and monitor progress towards elimination goals. To overcome this obstacle, the NMCP undertook a comprehensive data mapping project, aiming to consolidate and structure all activity data into a unified, accessible format.

and creating a robust and scalable data processing workflow for streamlined analysis, encompassing data cleaning, testing, merging, and manual review that can be used for previous and future data

The project successfully mapped data from main NMCP activities. Through a multi-layered data cleaning and integration process, incorporating automated scripts and manual review, the project established standardized master lists for villages, health facilities, and community health volunteers, ensuring consistent data referencing across all activities.

The resulting structured data model and workflow empowers the NMCP with the necessary process for streamlined analysis, encompassing data cleaning, testing, merging, visualizing and manual review that can be used for previous and future data

* **Track intervention coverage and identify gaps:** Visually map interventions geographically, pinpointing areas with insufficient coverage for targeted interventions.
* **Assess intervention effectiveness:** Analyze trends in malaria cases over time and by location, allowing for evidence-based evaluation of specific interventions.
* **Optimize resource allocation:** Strategically allocate resources based on identified needs and malaria hotspots, maximizing program efficiency.
* **Strengthen program monitoring:** Enhance routine monitoring processes with reliable data, enabling timely corrective actions and program adjustments.

## Introduction, Report 003

Malaria remains a significant public health threat in Yemen, imposing a substantial burden on healthcare systems and communities. The NMCP implements various interventions, including insecticide-treated net distribution, indoor residual spraying, and case management, to combat this disease. However, the effectiveness of these interventions hinges on accurate data collection, analysis, and decision-making.

Prior to this project, the NMCP’s data landscape was characterized by fragmentation and inconsistencies. Activity data resided in multiple, isolated systems with varying formats and coding conventions. This hindered comprehensive analysis, making it difficult to timely track intervention coverage, evaluate effectiveness, and target resources efficiently. Recognizing this critical limitation, the NMCP initiated a transformative data mapping project to unify and structure its activity data into a comprehensive and accessible format.

The project aimed to achieve the following objectives:

* **Consolidation of data from key sources:** Integrate data from a diverse range of activity sources, including spraying records, net distribution logs, and case reports, into a unified data repository.
* **Establishment and population of master lists:** Create and maintain standardized master lists for entities like villages, health facilities, and community health volunteers to ensure consistent data referencing.
* **Rigorous data cleaning and transformation:** Apply advanced data cleaning and transformation techniques to address inconsistencies, errors, and missing values, ensuring data quality and integrity.
* **Development of a unified data model:** Design a structured data model that captures essential information from all activities while maintaining logical relationships between entities, facilitating efficient data retrieval and analysis.
* **Workflow Automation:** Automate data cleaning, integration, and testing processes using workflow automation tools (Pentaho and Scripts) for increased efficiency and reduced human error.
* **Creation of data visualization tools and dashboards:** Develop intuitive data visualization tools and dashboards to enable insightful analysis and interpretation of the mapped data for informed decision-making.

By achieving these objectives, the NMCP aimed to unlock the true potential of its data, transforming it into a valuable asset for optimizing interventions, monitoring progress, and ultimately accelerating progress towards a malaria-free future in Yemen.

The following sections of this report will delve into the methodology employed, the achieved results, the broader implications for malaria control, and provide concrete recommendations for future data management and utilization within the NMCP.

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

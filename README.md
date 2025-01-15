# Employee-Survey-Data-Integration-and-Insights

---

## Table of Contents

1. [Project Description](#project-description)
2. [Technologies Used](#technologies-used)
3. [Features](#features)
4. [Setup Instructions](#setup-instructions)
5. [Managerial Insights](#managerial-insights)
6. [How to Use the Project](#how-to-use-the-project)
7. [Contributing](#contributing)
8. [License](#license)

## Project Description

This project involves the ingestion, transformation, and processing of employee survey data, with the goal of providing managerial insights through the use of Elasticsearch (ELK stack). The data pipeline integrates Kafka for real-time data streaming, Flink SQL for data processing, and Elasticsearch for storing and querying insights. The project provides valuable insights for improving employee engagement, performance, and overall organizational effectiveness.

## Technologies Used

- **Kafka**: For data streaming and ingestion.
- **Flink**: For real-time data processing and SQL-based transformations.
- **Elasticsearch (ELK Stack)**: For storing and querying data to derive insights.
- **Python**: For data transformation into JSON key-value pairs.
- **Docker**: For containerizing services such as Kafka and Elasticsearch.

## Features

- Real-time ingestion of employee and survey data via Kafka topics.
- Use of Flink SQL for data processing and merging of employee data with survey responses.
- Integration with Elasticsearch to store processed data and provide real-time search capabilities.
- Managerial insights based on the combined data, offering actionable recommendations for workforce improvement.

## Setup Instructions

### 1. Data Input to Consumer

The data is initially processed in the following steps:

- **Step 1**: Use the `convert.py` script to transform the raw data into a JSON format with key-value pairs. The script will prompt for a filename (e.g., `xyz.json`), which should be provided with the full path.
  
  ```bash
  python $HOME/Documents/fake/convert.py
  ```

- **Step 2**: Start Docker and use the `gen_sample.sh` script to pipe the key-value pair data to a Kafka topic (e.g., `mytest`) using `kafkacat`. The topic will be created if it does not exist.

  First, ensure Docker is running and then execute the following:

  ```bash
  ./start_flink_nodatagen.sh
  ```

  Wait for the services to start, then use the script to send the data:

  ```bash
  ./gen_sample.sh /home/ashok/Documents/gendata/rev_sample.json | kafkacat -b localhost:9092 -t mytest -K: -P
  ```

- **Step 3**: Open another terminal window and check if data is being fed into the Kafka topic using the following command:

  ```bash
  ./consumer.sh mytest
  ```

### 2. SQL Code for Creation, Merging, and Insertion

The following SQL code was used to create the necessary tables, perform data merging, and insert the data into Elasticsearch:

#### Creating Tables for Data Ingestion

```sql
CREATE TABLE employees (
    EmpID BIGINT,
    FirstName STRING,
    LastName STRING,
    Title STRING,
    Supervisor STRING,
    ADEmail STRING,
    BusinessUnit STRING,
    EmployeeStatus STRING,
    EmployeeType STRING,
    PayZone STRING,
    EmployeeClassificationType STRING,
    TerminationType STRING,
    DepartmentType STRING,
    Division STRING,
    State STRING,
    JobFunctionDescription STRING,
    GenderCode STRING,
    LocationCode BIGINT,
    RaceDesc STRING,
    MaritalDesc STRING,
    Performance_Score STRING,
    Current_Employee_Rating INT,
    EngagementScore INT,
    SatisfactionScore INT,
    WorkLifeBalanceScore INT,
    proctime AS PROCTIME()
) WITH (
    'connector' = 'kafka',
    'topic' = 'data1',
    'scan.startup.mode' = 'earliest-offset',
    'properties.bootstrap.servers' = 'kafka:9094',
    'format' = 'json'
);

CREATE TABLE survey_responses (
    EmpID BIGINT,
    EngagementScore INT,
    SatisfactionScore INT,
    WorkLifeBalanceScore INT
) WITH (
    'connector' = 'kafka',
    'topic' = 'survey',
    'scan.startup.mode' = 'earliest-offset',
    'properties.bootstrap.servers' = 'kafka:9094',
    'format' = 'json'
);
```

#### Merging Data from Employees and Survey Responses

```sql
CREATE VIEW employee_survey_view AS
SELECT 
    e.EmpID,
    e.FirstName,
    e.LastName,
    e.`StartDate`,
    e.Title,
    e.Supervisor,
    e.ADEmail,
    e.BusinessUnit,
    e.EmployeeStatus,
    e.EmployeeType,
    e.PayZone,
    e.EmployeeClassificationType,
    e.TerminationType,
    e.DepartmentType,
    e.Division,
    e.State,
    e.JobFunctionDescription,
    e.GenderCode,
    e.LocationCode,
    e.RaceDesc,
    e.MaritalDesc,
    e.Performance_Score,
    e.Current_Employee_Rating,
    s.EngagementScore,
    s.SatisfactionScore,
    s.WorkLifeBalanceScore
FROM 
    employees e
LEFT JOIN 
    survey_responses s
ON 
    e.EmpID = s.EmpID;
```

#### Inserting Data into Elasticsearch

```sql
CREATE TABLE employee_survey_es (
    EmpID BIGINT,
    FirstName STRING,
    LastName STRING,
    `StartDate` STRING,
    Title STRING,
    Supervisor STRING,
    ADEmail STRING,
    BusinessUnit STRING,
    EmployeeStatus STRING,
    EmployeeType STRING,
    PayZone STRING,
    EmployeeClassificationType STRING,
    TerminationType STRING,
    DepartmentType STRING,
    Division STRING,
    State STRING,
    JobFunctionDescription STRING,
    GenderCode STRING,
    LocationCode BIGINT,
    RaceDesc STRING,
    MaritalDesc STRING,
    Performance_Score STRING,
    Current_Employee_Rating INT,
    EngagementScore INT,
    SatisfactionScore INT,
    WorkLifeBalanceScore INT
) WITH (
    'connector' = 'elasticsearch-7',
    'hosts' = 'http://elasticsearch:9200',
    'index' = 'employee_survey',
    'document-id.key-delimiter' = '-',
    'format' = 'json',
    'sink.bulk-flush.max-actions' = '1'
);

INSERT INTO employee_survey_es
SELECT 
    e.EmpID,
    e.FirstName,
    e.LastName,
    e.`StartDate`,
    e.Title,
    e.Supervisor,
    e.ADEmail,
    e.BusinessUnit,
    e.EmployeeStatus,
    e.EmployeeType,
    e.PayZone,
    e.EmployeeClassificationType,
    e.TerminationType,
    e.DepartmentType,
    e.Division,
    e.State,
    e.JobFunctionDescription,
    e.GenderCode,
    e.LocationCode,
    e.RaceDesc,
    e.MaritalDesc,
    e.Performance_Score,
    e.Current_Employee_Rating,
    s.EngagementScore,
    s.SatisfactionScore,
    s.WorkLifeBalanceScore
FROM 
    employees e
LEFT JOIN 
    survey_responses s
ON 
    e.EmpID = s.EmpID;
```

## Managerial Insights

### **Objective**

To gain a deeper understanding of the workforce dynamics, identify key areas of strength and weakness, and provide actionable recommendations for improving employee engagement, performance, and overall organizational effectiveness.

---

## Dashboard


![em1](https://github.com/user-attachments/assets/c7ff0a9b-c53d-4eeb-98a0-542b24758282)

![em2](https://github.com/user-attachments/assets/4502ce05-3090-4b6c-9fe0-06c2c54b5c6a)

![em3](https://github.com/user-attachments/assets/afd8e501-effe-44a2-9247-8ad878144770)

### **Managerial Insights**

- **Workforce Composition**
  - The workforce is diverse with employees across various departments, job titles, and backgrounds.
  - The Production department has the highest employee count, indicating a significant focus on production activities.
  - The majority of employees are full-time, suggesting a stable workforce.
  - Gender diversity exists, although males constitute a slightly higher proportion.
  - Racial diversity is present, with White employees forming the largest group.

- **Employee Performance and Engagement**
  - Performance varies across divisions, with Division 1 showing the highest performance score.
  - There's a positive correlation between employee satisfaction and work-life balance, highlighting the importance of employee well-being.
  - Engagement scores fluctuate over time, indicating potential areas for improvement in engagement initiatives.
  - Engagement levels vary across supervisors, suggesting that leadership style and management practices play a significant role in employee engagement.

- **Key Areas of Strength**
  - A positive correlation between satisfaction and work-life balance indicates that prioritizing employee well-being can enhance job satisfaction and potentially improve performance.
  - The presence of different racial groups suggests a diverse workforce, which can bring diverse perspectives and ideas.
  - The high prevalence of "Production Technician II" roles suggests a strong foundation in production operations.

- **Key Areas of Weakness**
  - Division 4 exhibits the lowest performance score, requiring targeted interventions to improve performance and engagement.
  - Fluctuating engagement scores indicate potential issues that need to be addressed proactively.
  - The gender distribution leans towards males, suggesting potential areas for improvement in gender diversity and inclusion.

### **Actionable Recommendations**

- **Enhance Work-Life Balance:** Implement policies and programs that promote work-life balance, such as flexible work arrangements, generous leave policies, and wellness programs.
- **Improve Engagement in Division 4:** Conduct a thorough analysis of the factors contributing to lower performance in Division 4 and implement targeted interventions to improve engagement and performance. This could include:
  - **Supervisory training:** Provide training and coaching for supervisors to enhance their leadership skills, communication, and employee motivation.
  - **Employee surveys

:** Conduct regular employee surveys in Division 4 to gather feedback on their concerns, challenges, and suggestions for improvement.
  - **Performance reviews and goal setting:** Implement regular performance reviews and assist employees in setting clear, achievable goals.
- **Boost Overall Engagement:** Conduct regular employee surveys, pulse checks, and feedback mechanisms to monitor engagement levels across the organization. Address any emerging issues proactively.
- **Enhance Diversity and Inclusion:**
  - Implement initiatives to increase gender diversity, such as targeted recruitment efforts, mentorship programs, and unconscious bias training.
  - Create an inclusive work environment where all employees feel valued and respected.
- **Skill Development and Succession Planning:**
  - Analyze the job title word cloud to identify potential skill gaps and implement training programs to address these gaps.
  - Develop succession plans for key roles to ensure a smooth transition and continuity of operations.

## How to Use the Project

Follow the steps outlined in the **Setup Instructions** to get the project up and running. Once set up, the data will be ingested into Kafka, processed with Flink, and insights will be available in Elasticsearch. You can query the data in Elasticsearch to view insights, or modify the process as needed to suit your needs.

## Contributing

If you would like to contribute to this project, feel free to fork the repository, make changes, and submit a pull request. Contributions are welcome in the form of bug fixes, improvements, or additional features.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

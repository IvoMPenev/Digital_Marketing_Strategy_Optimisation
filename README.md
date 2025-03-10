![image](https://github.com/user-attachments/assets/49257762-d1db-4fa8-b99c-fc52f4b6ccc6)# Digital_Marketing_Strategy_Optimisation
Dataset
We'll be exploring the Google Merchandise Store dataset, containing data on website users from the US from October 1, 2016, to December 31, 2016. This dataset is sourced from the Google Merchandise Store, a real e-commerce platform offering Google-branded merchandise, including apparel, lifestyle products, and stationery from Google and its brands. While some sensitive fields have been obfuscated for privacy reasons, this dataset provides a unique opportunity to analyze actual customer behavior and trends in an e-commerce setting.
Disclaimer: for this analysis, certain fields within the dataset have been aggregated. The original dataset is available as a public dataset in Google BigQuery.

## Situation
The Google Marketing Team in the US was facing challenges in several key areas of their online business. Despite having a robust platform and a diverse range of products, the company was not fully leveraging its potential. With various strategies and channels in play, it was quite challenging to accurately assess the impact of different marketing initiatives and prove their effectiveness to stakeholders.
To address these challenges, the Google Marketing Team focused on gaining insighs into several key areas:
•	How do users interact with our e-commerce platform? Are there specific user behaviors or patterns that can inform our marketing strategies?
•	How effective are our promotional campaigns and discount strategies in driving sales? Do these initiatives lead to long-term customer retention or only short-term gains?
•	How does the user experience on our website and mobile app impact sales and customer engagement? Are there areas in need of improvement?
•	Which geographical markets or customer segments are most profitable? Are there emerging markets or segments that we should focus on?
•	How effective are our current strategies in acquiring new customers and retaining existing ones?
•	Which traffic channels are yielding the highest conversion rates? What are the underlying reasons for variations in conversion rates across different channels?

## Task
As the new marketing analyst at Google, my task involved conducting a detailed analysis of the e-commerce platform's performance. My main objective was to develop actionable strategies that would enhance overall performance. This included focusing on key areas such as customer acquisition, retention, monetization, and more efficient use of marketing channels. The ultimate aim was to significantly boost overall revenue.

## Action
In my role as a marketing analyst at Google, I used SQL to create a database and merge multiple datasets into a unified one for in-depth analysis, followed by thorough data preprocessing, including cleansing to ensure accuracy and reliability. I then conducted advanced analyses using advanced string, date and window functions to gain deeper insights into the website's performance, focusing on user interactions and the effectiveness of our e-commerce strategies. Additionally, I analyzed various web traffic sources to identify the most effective channels for driving sales and conversions, turning data into actionable business intelligence.

## Set Up
In MySQL Workbench, we’ll create a new database called gms_project. This database will be used to store tables and data as part of our project.

![image](https://github.com/user-attachments/assets/8507b4e6-6219-4f86-8dcc-461dd1fdccf2)
## Combining Datasets
Our first task is to create a unified view of the data across the three months. The combined dataset will serve as the base of our analysis.
CREATE TABLE gms_project.data_combined AS (

	SELECT * FROM gms_project.data_10
	UNION ALL 
	SELECT * FROM gms_project.data_11
	UNION ALL
	SELECT * FROM gms_project.data_12;

## Data Exploration
To get a sense of the data we're working with, we’ll take a look at the first few rows of the table. This allows us to see what columns are available and the kind of data each column contains.

![image](https://github.com/user-attachments/assets/09e4982f-e075-43ee-a7ba-41fdf44b1fcc)

Column Overview: The dataset contains rows with identifiers like fullvisitorId and visitId, along with the date of visits and details of traffic sources in the channelGrouping, source, and medium columns. It offers insights into user engagement through metrics such as visits, pageviews, time on website, and bounces.
Additionally, it includes e-commerce data (transactions and revenue), user-specific information (operating system, mobile device usage, device type), and geographical data (region, country, continent, subcontinent). Some fields, like adcontent, are missing data. 
Row Overview: At first glance, it seems that each row represents a
single session on the website. The visitid column appears to be a
unique identifier for each session, and therefore, needs closer investigation.
First, we’ll examine the visitid column for null values. Ensuring
there are no NULL values in this key column is essential for the integrity of our dataset.

![image](https://github.com/user-attachments/assets/e8b244e8-146b-4522-88c5-b797f14042d1)
It appears that there are no NULL values in the visited column,
indicating that each session captured in the data has been assigned an identifier. The absence of NULL values in this key column is a positive sign for data integrity.
Next, we'll check for duplicate values in the visited column.
dentifying duplicates is important as they can significantly impact the accuracy of our analysis. We'll determine whether these duplicates represent valid data repetitions or if they need to be addressed.

![image](https://github.com/user-attachments/assets/3e4b8021-cee9-4dd9-a9cd-3f0807875bcf)

The visitid column in the dataset, while seemingly unique, does not serve as a unique identifier for each session. This is because visitid represents the timestamp when a visit or session begins. Since multiple visitors can start their sessions at the same exact time, visitid alone is not sufficient to uniquely identify each session. To create a unique identifier for each session, we need to combine visitId with another column, fullvisitorId. The fullvisitorId column uniquely identifies each visitor to the website. By concatenating fullvisitorId and visitid, we can create a new identifier that is unique for each session. This new identifier will ensure that each session is distinctly recognized, even if multiple sessions start at the same time.

![image](https://github.com/user-attachments/assets/4bada90e-ce34-4147-a93f-a063319a04af)

From the analysis of the data, it appears that we still have two duplicate entries. This is likely due to how sessions are tracked around midnight. In many web analytics systems, a visitor's session is reset at midnight. This means that if a visitor is active on the website across midnight, their activity before and after midnight is counted as two separate sessions. However, if the visitid is based on the start time of the session, then the visitor will have the same sessions. When we concatenate this visitid with the fullvisitorid, the resulting unique_session_id will be the same for both sessions. Therefore, despite being on the website across two different sessions (before and after midnight), the visitor appears in our data with the same unique_session_id for both sessions across Let’s examine one example.

Note: Our dataset timestamps are in UTC (Coordinated Universal Time), the main time standard used globally, which remains constant year-round. As our analysis focuses on US data, we'll convert these timestamps to PDT (Pacific Daylight Time). PDT applies to the US and Canadian Pacific Time Zone during summer months, operating 7 hours behind UTC (UTC-7). In winter, this region switches to PST (Pacific Standard Time), which is 8 hours behind UTC.

![image](https://github.com/user-attachments/assets/f11ca56e-9ea7-4ea4-8879-774f6a7b3223)

In our analysis, we acknowledge this scenario and have decided to treat these two sessions as a single continuous session, maintaining the same unique_session_id. This approach aligns with our analytical objectives and simplifies our dataset. Therefore, we won't modify the session tracking mechanism to separate these instances into distinct sessions.

Business Insights
Website Engagement by Day
Now that we have a basic understanding of our dataset, we're ready to delve into a more detailed analysis. First, we'll explore daily web traffic to gain insights int overall website. This initial step will reveal key traffic patterns and trends in visitor behavior.
Note: For simplicity, our analysis will use UTC time for all timestamps, avoiding the complexities of time zone conversions and Daylight Saving Time adjustments.

![image](https://github.com/user-attachments/assets/d2e28906-5f46-4f66-a81e-5ae3e9f8bbad)

We observe an uptick in web traffic as we approach the holiday season in December. In addition, web traffic consistently peaks during weekdays and tapers off during weekends. To better illustrate this trend, we'll extract the name of the day from the visit date.
We notice a variation in visitor numbers, with web traffic peaking mid- week, particularly on Tuesdays and Wednesdays, and then declining over the weekend. This pattern indicates that user engagement on the website fluctuates throughout the week. This insight can be crucial for planning content updates and marketing initiatives.
	
### Website Engagement & Monetization by Day

We can delve deeper into this trend by examining conversion rates to determine if these visits result in transactions. This analysis will provide a clearer understanding of how effectively web traffic is translating into actual sales.

![image](https://github.com/user-attachments/assets/7eec7712-a5d9-4001-97b9-9e6614429789)

While Tuesdays see the most website visits, it's interesting to note that 
Mondays have the highest conversion rate. In contrast, weekends 
experience a substantial drop in conversions, indicating different visitor 
behaviors and purchasing patterns between weekdays and weekends. 

Website Engagement & Monetization by Device
We can further refine our analysis by examining the data by device type, which will reveal variations in conversion rates and visits across desktops, tablets, and smartphones. This device-specific insight is key for optimizing website design and marketing for each device category.
Note: in the dataset, revenue is expressed in a scaled format, where the actual value is multiplied by 10^6. For instance, a revenue of $2.40 is recorded as 2,400,000. To interpret these figures accurately, we’ll divide them by 10^6 to get the original dollar amount.

![image](https://github.com/user-attachments/assets/93c2a8a4-8b07-4ee7-9523-36250f351137)

![image](https://github.com/user-attachments/assets/2e38e1ed-8f6e-45fa-8818-43cac78ff6c5)  ![image](https://github.com/user-attachments/assets/8f29b331-cf71-4092-b19d-5f250153296d)


While ~25% of sessions originate from mobile devices, only 5% of revenue is generated through them. This significant discrepancy suggests a need to optimize the mobile shopping experience.
Marketing strategies should focus on enhancing mobile usability,  streamlining the checkout process, and tailoring mobile-specific   promotions. Considering the significant number of users who shop on their mobile devices during commutes or workday breaks, a seamless mobile experience on our e-commerce platform is crucial. To further tap into this growing user base, Google might also consider developing a dedicated mobile app, which could substantially increase revenue from mobile users.

### Website Engagement & Monetization by Region


![image](https://github.com/user-attachments/assets/039f084a-53ac-464e-b48e-d4738958504f)























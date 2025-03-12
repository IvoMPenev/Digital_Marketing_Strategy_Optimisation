# Dataset
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
```
CREATE DATABASE `gms_project`;
```


![image](https://github.com/user-attachments/assets/8507b4e6-6219-4f86-8dcc-461dd1fdccf2)
## Combining Datasets
Our first task is to create a unified view of the data across the three months. The combined dataset will serve as the base of our analysis.
CREATE TABLE gms_project.data_combined AS (
```
SELECT * FROM gms_project.data_10
UNION ALL 
SELECT * FROM gms_project.data_11
UNION ALL
SELECT * FROM gms_project.data_12;
```
## Data Exploration
To get a sense of the data we're working with, we’ll take a look at the first few rows of the table. This allows us to see what columns are available and the kind of data each column contains.
```
SELECT *
FROM gms_project.data_combined
LIMIT 5;
```

![image](https://github.com/user-attachments/assets/09e4982f-e075-43ee-a7ba-41fdf44b1fcc)

Column Overview: The dataset contains rows with identifiers like fullvisitorId and visitId, along with the date of visits and details of traffic sources in the channelGrouping, source, and medium columns. It offers insights into user engagement through metrics such as visits, pageviews, time on website, and bounces.
Additionally, it includes e-commerce data (transactions and revenue), user-specific information (operating system, mobile device usage, device type), and geographical data (region, country, continent, subcontinent). Some fields, like adcontent, are missing data. 

Row Overview: At first glance, it seems that each row represents a
single session on the website. The visitid column appears to be a
unique identifier for each session, and therefore, needs closer investigation.
First, we’ll examine the visitid column for null values. Ensuring
there are no NULL values in this key column is essential for the integrity of our dataset.
```
SELECT  
	COUNT(*) AS total_rows, 
	COUNT(visitid) AS non_null_rows 
FROM gms_project.data_combined;
```

![image](https://github.com/user-attachments/assets/e8b244e8-146b-4522-88c5-b797f14042d1)

It appears that there are no NULL values in the visited column,
indicating that each session captured in the data has been assigned an identifier. The absence of NULL values in this key column is a positive sign for data integrity.
Next, we'll check for duplicate values in the visited column.
dentifying duplicates is important as they can significantly impact the accuracy of our analysis. We'll determine whether these duplicates represent valid data repetitions or if they need to be addressed.
```
SELECT
	visitid,
	COUNT(*) as total_rows
FROM gms_project.data_combined
GROUP BY 1
HAVING COUNT(*) > 1
LIMIT 5;
```

![image](https://github.com/user-attachments/assets/3e4b8021-cee9-4dd9-a9cd-3f0807875bcf)

The visitid column in the dataset, while seemingly unique, does not serve as a unique identifier for each session. This is because visitid represents the timestamp when a visit or session begins. Since multiple visitors can start their sessions at the same exact time, visitid alone is not sufficient to uniquely identify each session. To create a unique identifier for each session, we need to combine visitId with another column, fullvisitorId. The fullvisitorId column uniquely identifies each visitor to the website. By concatenating fullvisitorId and visitid, we can create a new identifier that is unique for each session. This new identifier will ensure that each session is distinctly recognized, even if multiple sessions start at the same time.
```
SELECT
	CONCAT(fullvisitorid, '-', visitid) AS unique_session_id,
	COUNT(*) as total_rows
FROM gms_project.data_combined
GROUP BY 1
HAVING COUNT(*) > 1
LIMIT 5;
```

![image](https://github.com/user-attachments/assets/7ccd7278-70aa-4ba3-863a-9be54999027e)

From the analysis of the data, it appears that we still have two duplicate entries. This is likely due to how sessions are tracked around midnight. In many web analytics systems, a visitor's session is reset at midnight. This means that if a visitor is active on the website across midnight, their activity before and after midnight is counted as two separate sessions. However, if the visitid is based on the start time of the session, then the visitor will have the same visitid for both sessions. When we concatenate this visitid with the fullvisitorid, the resulting unique_session_id will be the same for both sessions. Therefore, despite being on the website across two different sessions (before and after midnight), the visitor appears in our data with the same unique_session_id for both sessions. Let’s examine one example.

*Note: Our dataset timestamps are in UTC (Coordinated Universal Time), the main time standard used globally, which remains constant year-round. As our analysis focuses on US data, we'll convert these timestamps to PDT (Pacific Daylight Time). PDT applies to the US and Canadian Pacific Time Zone during summer months, operating 7 hours behind UTC (UTC-7). In winter, this region switches to PST (Pacific Standard Time), which is 8 hours behind UTC.*

```
SELECT
	CONCAT(fullvisitorid, '-', visitid) AS unique_session_id,
	FROM_UNIXTIME(date) - INTERVAL 8 HOUR AS date,
	COUNT(*) as total_rows
FROM gms_project.data_combined
GROUP BY 1,2
HAVING unique_session_id = "4961200072408009421-1480578925"
LIMIT 5;
```

![image](https://github.com/user-attachments/assets/46ce1caf-3f42-43dd-b111-053b424cce73)

In our analysis, we acknowledge this scenario and have decided to treat these two sessions as a single continuous session, maintaining the same unique_session_id. This approach aligns with our analytical objectives and simplifies our dataset. Therefore, we won't modify the session tracking mechanism to separate these instances into distinct sessions.

Business Insights

Website Engagement by Day

Now that we have a basic understanding of our dataset, we're ready to delve into a more detailed analysis. First, we'll explore daily web traffic to gain insights into overall website performance. This initial step will reveal key traffic patterns and trends in visitor behavior.

*Note: For simplicity, our analysis will use UTC time for all timestamps, avoiding the complexities of time zone conversions and Daylight Saving Time adjustments.*
```
SELECT 
    date,
    COUNT(DISTINCT unique_session_id) AS sessions
FROM (
	SELECT
		DATE(FROM_UNIXTIME(date)) AS date,
		CONCAT(fullvisitorid, '-', visitid) AS unique_session_id
	FROM gms_project.data_combined
	GROUP BY 1,2
) t1
GROUP BY 1
ORDER BY 1;
```


![image](https://github.com/user-attachments/assets/9f98e09a-90b8-4ade-99bf-2428083cfe15)


![image](https://github.com/user-attachments/assets/eb5d403c-989f-4756-b752-4baea2fee344)


We observe an uptick in web traffic as we approach the holiday season in December. In addition, web traffic consistently peaks during weekdays and tapers off during weekends. To better illustrate this trend, we'll extract the name of the day from the visit date.
```
SELECT 
    DAYNAME(date) AS weekday,
    COUNT(DISTINCT unique_session_id) AS sessions
FROM (
	SELECT
		DATE(FROM_UNIXTIME(date)) AS date,
        	CONCAT(fullvisitorid, '-', visitid) AS unique_session_id
	FROM gms_project.data_combined
	GROUP BY 1,2
) t1
GROUP BY 1
ORDER BY 2 DESC;
```

![image](https://github.com/user-attachments/assets/be84325b-1669-4a97-a532-32e6db58eb44)

We notice a variation in visitor numbers, with web traffic peaking mid-week, particularly on Tuesdays and Wednesdays, and then declining over the weekend. This pattern indicates that user engagement on the website fluctuates throughout the week. This insight can be crucial for planning content updates and marketing initiatives.

Website Engagement & Monetization by Day

We can delve deeper into this trend by examining conversion rates to determine if these visits result in transactions. This analysis will provide a clearer understanding of how effectively web traffic is translating into actual sales.
```
SELECT 
    DAYNAME(date) AS weekday,
    COUNT(DISTINCT unique_session_id) AS sessions,
    SUM(converted) AS conversions,
    (SUM(converted) / COUNT(DISTINCT unique_session_id)) * 100 AS conversion_rate
FROM (
    SELECT
        DATE(FROM_UNIXTIME(date)) AS date,
        CASE 
            WHEN transactions >= 1 THEN 1
            ELSE 0 
        END AS converted,
        CONCAT(fullvisitorid, '-', visitid) AS unique_session_id
    FROM gms_project.data_combined
) t1
GROUP BY weekday
ORDER BY sessions DESC;
```

![image](https://github.com/user-attachments/assets/645b4228-a585-4b68-ae54-56aa94391873)

While Tuesdays see the most website visits, it's interesting to note that Mondays have the highest conversion rate. In contrast, weekends experience a substantial drop in conversions, indicating different visitor behaviors and purchasing patterns between weekdays and weekends.

Website Engagement & Monetization by Device

We can further refine our analysis by examining the data by device type, which will reveal variations in conversion rates and visits across desktops, tablets, and smartphones. This device-specific insight is key for optimizing website design and marketing for each device category.

*Note: in the dataset, revenue is expressed in a scaled format, where the actual value is multiplied by 10^6. For instance, a revenue of $2.40 is recorded as 2,400,000. To interpret these figures accurately, we’ll divide them by 10^6 to get the original dollar amount.*

```
SELECT
    deviceCategory,
    COUNT(DISTINCT unique_session_id) AS sessions,
    (COUNT(DISTINCT unique_session_id) / SUM(COUNT(DISTINCT unique_session_id)) OVER ()) * 100 AS sessions_percentage,
    SUM(transactionrevenue) / 1e6 AS revenue,
    (SUM(transactionrevenue) / SUM(SUM(transactionrevenue)) OVER ()) * 100 AS revenue_percentage
FROM (
    SELECT
        deviceCategory,
        transactionrevenue,
        CONCAT(fullvisitorid, '-', visitid) AS unique_session_id
    FROM gms_project.data_combined
) t1
GROUP BY deviceCategory;
```

![image](https://github.com/user-attachments/assets/f1785299-e6f3-483f-b16d-696bfcc4de3b)

![image](https://github.com/user-attachments/assets/693acdcc-9147-429d-b75d-ffce0b42cb37)

![image](https://github.com/user-attachments/assets/41d6b7b9-582b-4101-b354-db03b6a27d7f)


While ~25% of sessions originate from mobile devices, only 5% of revenue is generated through them. This significant discrepancy suggests a need to optimize the mobile shopping experience. Marketing strategies should focus on enhancing mobile usability, streamlining the checkout process, and tailoring mobile-specific promotions. Considering the significant number of users who shop on their mobile devices during commutes or workday breaks, a seamless mobile experience on our e-commerce platform is crucial. To further tap into this growing user base, Google might also consider developing a dedicated mobile app, which could substantially increase revenue from mobile users.

Website Engagement & Monetization by Region

We can analyse these numbers further to determine if there are regional differences affecting mobile device usage and revenue generation. This deeper dive will help us understand whether certain regions have higher mobile engagement or sales, guiding targeted marketing strategies and region-specific optimizations.
```
SELECT
    deviceCategory,
    region,
    COUNT(DISTINCT unique_session_id) AS sessions,
    (COUNT(DISTINCT unique_session_id) / SUM(COUNT(DISTINCT unique_session_id)) OVER ()) * 100 AS sessions_percentage,
    SUM(transactionrevenue) / 1e6 AS revenue,
    (SUM(transactionrevenue) / SUM(SUM(transactionrevenue)) OVER ()) * 100 AS revenue_percentage
FROM (
    SELECT
        deviceCategory,
        CASE 
            WHEN region = '' OR region IS NULL THEN 'NA'
            ELSE region 
        END AS region,
        transactionrevenue,
        CONCAT(fullvisitorid, '-', visitid) AS unique_session_id
    FROM gms_project.data_combined
    WHERE deviceCategory = 'mobile'
) t1
GROUP BY deviceCategory, region
ORDER BY sessions DESC;
```

![image](https://github.com/user-attachments/assets/82890db1-40e9-44d9-b19e-33e4d59e7c3f)

The data shows that while only 1% of mobile sessions are from Washington, they contribute to 11% of revenue. Similarly, Illinois sees 3% of sessions but accounts for 9% of revenue. This suggests an untapped opportunity, as these regions have higher conversion rates or transaction values despite fewer sessions. Focusing marketing efforts on these regions could potentially increase revenue, leveraging their higher purchasing effectiveness. Potential approaches could include targeted marketing campaigns, localized promotions, or even exploring the reasons behind the higher conversion rates, such as product preferences or purchasing power.
One limitation in our analysis arises from the fact that some mobile sessions in the dataset are not mapped to any specific region. This means that for a subset of mobile sessions, the region field is either left blank or marked as NULL, indicating that the geographic location of these users is unknown or not recorded. Addressing this issue with the Data Engineering team could be vital for ensuring more accurate and comprehensive data for future analyses.

Website Retention

Next, we’ll examine website retention, specifically focusing on whether users are new or returning. This will provide insights into user loyalty and the effectiveness of strategies in encouraging repeat visits. Typically, having around 50-70% new users and 30-50% returning users is considered a good balance.
```
SELECT
    CASE 
        WHEN newVisits = 1 THEN 'New Visitor'
        ELSE 'Returning Visitor'
    END AS visitor_type,
    COUNT(DISTINCT fullVisitorId) AS visitors,
    (COUNT(DISTINCT fullVisitorId) / SUM(COUNT(DISTINCT fullVisitorId)) OVER ()) * 100 AS visitors_percentage
FROM gms_project.data_combined
GROUP BY visitor_type;
```

![image](https://github.com/user-attachments/assets/5434bc1c-03d3-4d09-9ad1-d095361f83aa)

Interestingly, about 80% of users visit the website only once. This statistic suggests a need for better incentives or value propositions to encourage repeat visits, presenting a major opportunity for enhanced retention strategies such as personalized marketing, loyalty programs, or targeted retargeting campaigns. In contrast, the substantial influx of new visitors reflects successful marketing efforts in brand awareness, effectively attracting people to the site initially. Key factors like a user-friendly interface, engaging content, and smooth navigation play a crucial role here.

Website Acquisition

Building on this analysis, the next logical step is to calculate the bounce rate. This measure is key as it indicates the proportion of visitors who exit the site after viewing only one page and is often used to evaluate the effectiveness of acquisition strategies. A high bounce rate can indicate that the landing page or the initial content isn't meeting the expectations of visitors or isn't engaging enough to encourage further exploration of the site or purchases. Understanding the bounce rate offers valuable insights into the site's initial engagement success with its audience.
```
SELECT
    COUNT(DISTINCT unique_session_id) AS sessions,
    SUM(bounces) AS bounces,
    (SUM(bounces) / COUNT(DISTINCT unique_session_id)) * 100 AS bounce_rate
FROM (
    SELECT
        bounces,
        CONCAT(fullvisitorid, '-', visitid) AS unique_session_id
    FROM gms_project.data_combined
) t1;
```

![image](https://github.com/user-attachments/assets/248a7556-54ec-4863-9559-46e8736238fa)

A bounce rate within the 20-40% range is generally seen as effective engagement. However, to truly understand visitor behavior and optimize engagement strategies, it's important to analyze the bounce rate by individual channels. This deeper analysis will enable us to tailor our strategies to specific audiences, refine our understanding of user interactions, and allocate marketing resources more efficiently.

Website Acquisition by Channel

Moving forward, we'll delve into the bounce rate by channel to gain a clearer picture of the effectiveness of various marketing strategies. This will also shed light on user engagement across diverse traffic sources, providing valuable insights for optimizing our outreach efforts.
```
SELECT
    channelGrouping,
    COUNT(DISTINCT unique_session_id) AS sessions,
    SUM(bounces) AS bounces,
    (SUM(bounces) / COUNT(DISTINCT unique_session_id)) * 100 AS bounce_rate
FROM (
    SELECT
        channelGrouping,
        bounces,
        CONCAT(fullvisitorid, '-', visitid) AS unique_session_id
    FROM gms_project.data_combined
) t1
GROUP BY channelGrouping
ORDER BY sessions DESC;
```

![image](https://github.com/user-attachments/assets/7ac1cb9e-635b-4419-acc6-01e39f636ec5)

Organic Search, Paid Search, and Display exhibit bounce rates of 30-40%, indicating effective visitor engagement and an ability to encourage exploration beyond the landing page. Referral shows the lowest bounce rate, marking it as particularly healthy. However, Direct, Social, and Affiliates, with bounce rates of 40-50%, highlight areas where there is potential for improvement.

Website Acquisition & Monetization by Channel

Building on our analysis of visits and bounce rates by channel, we'll next include key metrics like time on site, revenue, and conversion rate. This broader evaluation will give us a fuller view of each channel's performance, encompassing not only traffic volume but also user engagement quality, revenue generation efficiency, and overall conversion impact.
```
SELECT
    channelGrouping,
    COUNT(DISTINCT unique_session_id) AS sessions,
    SUM(bounces) AS bounces,
    (SUM(bounces) / COUNT(DISTINCT unique_session_id)) * 100 AS bounce_rate,
    SUM(pageviews) / COUNT(DISTINCT unique_session_id) AS avg_pagesonsite,
    SUM(timeonsite) / COUNT(DISTINCT unique_session_id) AS avg_timeonsite,
    SUM(CASE WHEN transactions >= 1 THEN 1 ELSE 0 END) AS conversions,
    (SUM(CASE WHEN transactions >= 1 THEN 1 ELSE 0 END) / COUNT(DISTINCT unique_session_id)) * 100 AS conversion_rate,
    SUM(transactionrevenue) / 1e6 AS revenue
FROM (
    SELECT
        channelGrouping,
        bounces,
        pageviews,
        timeonsite,
        transactions,
        transactionrevenue,
        CONCAT(fullvisitorid, '-', visitid) AS unique_session_
```

![image](https://github.com/user-attachments/assets/c7bfd391-d158-4c4f-9b07-d59393896ac7)

Referral leads with the lowest bounce rate and highest conversion at 7%, driving strong revenue, making it a prime candidate for expanded partnerships. Organic Search, with the most sessions and a solid 32% bounce rate, shows robust SEO efficacy and good conversion potential. Direct traffic, while high at a 46% bounce rate, has moderate conversions, suggesting a need for more personalized engagement. Social media, despite high traffic, suffers from the lowest conversions, calling for more targeted campaigns. Paid Search and Display, both with 34% bounce rates, demonstrate moderate visitor retention but require improved targeting for better conversions. Lastly, Affiliates, with the highest bounce rate and lowest conversions, need a thorough evaluation of partner quality.

Results

In conclusion, let's recap our recommendations below to summarize the key strategies and insights derived from our analysis.
Seasonal and Holiday Campaigns: During key periods, such as the December holidays, create themed promotions and exclusive holiday deals. Develop holiday guides, offer limited-time discounts, and run festive-themed marketing campaigns to attract shoppers. Replicating this strategy in other months could potentially lead to a 25% uplift in revenue, akin to what is typically seen in December.

Maximize Monday Conversions: With Mondays having the highest conversion rates but lower traffic, aim to boost Monday traffic by at least 10% to reach Tuesday levels. Launch start-of-the-week promotions and exclusive deals, and actively promote these through targeted email and social media campaigns early in the week.

Targeted Weekday Promotions: Leverage the high traffic on Tuesdays and Wednesdays with special promotions and deals. Tailor your email marketing and social media campaigns to coincide with these peak days, ensuring promotions reach users when they are most active online.

Weekend Engagement Strategies: Use retargeting ads to re-engage weekday visitors with special weekend promotions. Implement loyalty programs offering rewards for weekend shopping to boost off-peak traffic and sales by 15%.
Enhance Mobile Experience: With 25% of sessions but only 5% of revenue coming from mobile, and considering that many users shop during commutes or workday breaks, focus on improving the mobile shopping experience. This could involve optimizing the mobile site's user interface, offering mobile-specific deals, or enhancing mobile payment options.

Focus on High-Value Regions: Address the disproportionate revenue contribution from Washington and Illinois. Develop targeted marketing strategies for these regions, possibly with localized offers or campaigns, to capitalize on their higher spending patterns.

Strengthen User Engagement and Retention: With 80% one-time visitors, target a 10% increase in repeat visits by enhancing the user experience for first-time visitors through personalized content, retargeting campaigns, and loyalty programs. Focus on engaging direct traffic with personalized strategies to reduce its high bounce rate.

Optimize Referral and Organic Channels: Due to their low bounce rate and high conversion, aim to increase referral traffic by 20% through expanded partnerships. Continue to improve SEO for Organic Search, which shows robust session numbers and a good potential for conversions.

Revamp Social Media and Paid Channels: Redesign social media campaigns for higher engagement and conversion. Improve targeting in Paid Search and Display advertising to reduce the 34% bounce rate and enhance visitor retention.

Reevaluate and Enhance Affiliate Strategies: Thoroughly assess affiliate partnerships, focusing on the quality and relevance to address their high bounce rate and low conversions. Restructure or terminate underperforming affiliations to improve efficiency.

These recommendations aim to capitalize on identified trends and insights, maximizing revenue opportunities throughout the year.

Next Steps

Moving forward, our analysis could delve into several key areas to further enhance our understanding and strategy:

Peak Hours Analysis: We could investigate the specific hours of the day that yield the highest number of website visits and conversions. This would help us optimize our engagement strategies to target peak activity times.

Regional Performance: We could examine regions with a high number of visits but low conversion rates to identify potential barriers to conversion, and then tailor our regional strategies accordingly.

Buyer Behavior Segmentation: We could segment and analyze two distinct customer groups: bulk buyers versus individual purchasers. Understanding their differing motivations and behaviors can inform targeted marketing and sales tactics.

Transaction Value Assessment: We could evaluate the average value of each transaction. A low average order size or value might indicate potential revenue losses, underscoring the need for strategic adjustments.

These focused analyses would provide deeper insights, enabling us to refine our strategies and drive more effective and targeted marketing initiatives.














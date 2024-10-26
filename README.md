# Google Bikeshare Analysis
## Background and Overview
This project is the capstone for the Google Data Analytics Certificate. This is Case Study 1, which deals with a bike-share company. From the description of the case study:

>You are a junior data analyst working in the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, your team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, your team will design a new marketing strategy to convert casual riders into annual members. But first, Cyclistic executives must approve your recommendations, so they must be backed up with compelling data insights and professional data visualizations.

The case study gives this information about the company:
  * 5,824 bicycles and 692 docking stations across Chicago
  * about 30% of users use the bikes to commute to work each day
  * single-ride passes, full-day passes, and annual memberships available
  * annual members are much more profitable than casual riders

The company's goal and reason for the analysis: design marketing strategies aimed at converting casual riders into annual members.

The data for this project is available [here](https://divvy-tripdata.s3.amazonaws.com/index.html).
I'm using one year of data from 2022-05 until 2023 - 04.
The total size of this year's worth of data is over 2 gigabytes, so I have done the majority of the analysis in SQL. I originally used SQLite, but I have reformatted my analysis for a cleaner writeup in an R markdown file.  I have also created visualizations with R and an interactive visualization of important bike routes in Tableau, viewable [here](https://public.tableau.com/app/profile/john.parchment/viz/GoogleBikeShareAnalysis/Dashboard1?publish=yes).

## Data Structure
The data is organized into csv files by month. For this analysis, I am looking at trends over one year, so I've combined 12 months of cvs files into one SQL database in R. I performed several SQL queries to check for data quality, and did some basic data cleaning, viewable in the [complete R markdown file](https://taylor-parchment.github.io/google-bikeshare-analysis/).

This is the structure of the data:

<img width="240" alt="image" src="https://github.com/user-attachments/assets/b7943475-31b6-4e5b-9f4c-1238047aac7f">

where started_at and ended_at are date-time values.

## Summary and Insights

- As one might expect, the biggest difference between casual riders and members is definitely the reason for using the bikes. Many members commute to work around 8 am, shown by the significant number of rides around that time. However, casual riders, using these bikes for recreation, most commonly rent bikes in the afternoon hours, peaking at around 5 pm.

 ![image](https://github.com/user-attachments/assets/f4e07dff-7b8f-4c1a-8526-5541c41a0929)


- All riders prefer to ride in spring and summer months, with peak ridership in July. However, members consistently use their bikes in winter months too, maintaining around 150,000 rides per winter month. Casual riders drop significantly in winter, to below 50,000 per month. All riders take longer trips in warmer months, but casual riders ride particularly longer in May. Casual riders take most trips during the weekend, while members take more trips during weekdays.

![image](https://github.com/user-attachments/assets/3a48cf1a-6112-4e8b-b521-0f030100c5f7)
![image](https://github.com/user-attachments/assets/e5b35a2f-3769-4507-bc30-f468b72c0c05)
![image](https://github.com/user-attachments/assets/6e932b3e-2c53-470e-b6d0-5df2227a6646)

- There isn’t a significant difference in distance covered by members or casual riders, however casual riders more often return to their station of origin, probably only riding for the sake of it and not to go somewhere.

## Recommendations

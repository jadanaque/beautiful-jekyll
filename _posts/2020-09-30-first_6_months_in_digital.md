---
layout: post
title: My First 6 Months In Digital
subtitle: My Journey Working in Digital at L'Oréal
tags: [professional, personal]
---

After 6 years working in Operations, at L’Oréal Peru, this year I transitioned to Digital. Last month (August) was my 6th month as Digital Data Manager, so it occurred to me to share some of the things I’ve been able to work on and learn so far because, although I’ve been working with data for a long time and taking many courses in Data Science and Machine Learning (ML), Digital is a whole new world, with tons of data from too many sources and new domains to explore (E-Commerce, Social Media, etc.), all of which I consider worth perpetuating noting here, in a small article.

For those who don’t know me, in Operations my work was more related to Data Governance and Master Data Management, with some project management and analysis along the way. Now, in Digital, it is more Data Science related, also with some project management which is always necessary when you grow in your career.

### My First Day

My first day I was told about the necessity to organize and visualize all our Sell-Out data in E-Commerce, from e-retailers to marketplaces. I naively asked for the deadline, the answer of which was “yesterday”. So, I immediately looked for the people involved with these data sources, trying to understand the magnitude of the problem and how to approach it. After some days I decided to start with the most difficult datasets, using [R](https://www.r-project.org/) to automate the processing. Then I moved to Power BI, using the power of its Power Query editor to process the rest of the datasets. Next, I aggregated everything into a huge table of sales data (fact table). Finally, I added some more necessary tables into the data model (dim tables) and built a nice-looking dashboard that is currently used and updated weekly. Everything in around a month! 

Of course I am avoiding many details here, but believe me, it got me some headaches in the process --totally worth it because as far as I know it is being considered as a benchmark to build something similar for the rest of the countries in Latam.

### Hallway Conversations

In one occasion, I heard a couple of friends discussing some problems with our websites. Apparently, some Call-To-Action (CTA) buttons were not working, and the review of every button for every product -and in some cases for every tone- had to be made manually, one by one.

An idea immediately came to me: [Web Scraping](https://es.wikipedia.org/wiki/Web_scraping). I scheduled a meeting to discuss the problem and see if it was worth investing the time in developing a web scraper or paying for an existing tool. I also had to explain my idea to them, which initially had the only goal of automating this process of reviewing the URLs in some buttons in our brand websites. We then had to discuss it with our manager, who gave us his approval -now it was my turn to build an MVP.

To make the story short, I built the Web Scraper from scratch, using R as the programming language –later realizing, after some difficulties, that maybe I should have used Python. I added some more very useful functionalities than originally required and wrapped it nicely in a web application so the users can easily use it.

It was a hit! Now it is being discussed regionally whether to use what I built as a tool for all the countries in the region (Latam) or to pay for an existing tool that is currently used in other regions at L’Oréal but that gives more functionality. Of course, if the tool I built is picked, an external provider will be needed in order to productionize it and give the support needed for the region.

### Transactional Data

We have a partner that gives us access to the transactional data generated in their platform. There are many formal agreements in order to make this possible. Now the problem is to integrate this source of data to our CRM (Salesforce) and start working in possible strategies for mailing with this database.

We are still working in the integration between the APIs, but in order to accelerate things I am currently in charge of analyzing and modeling this data in order to get accurate segments/clusters we can use to improve our sales performance.

### Trainings Taken and Given

I am in charge of all the data we have in Digital and what we can do with it, but in order to make sense of it and really seize its potential I’ve had to learn and take trainings in Digital Marketing, E-Commerce, Social Media, CRM, and much more. And I am not done with it. I am aware there is still a lot more to learn in order to contribute more with the company.

Lucky me, L’Oréal has a lot of resources to foster employees learning and development. I have taken a lot of workshops, masterclasses and online lessons. And we have access to resources like General Assembly and Coursera for self-training. For example, I am currently in the middle of a personalized track given by the General Assembly learning platform in order to get certified in Digital Marketing (the CM1 certification).

Also, I’ve had to get into the details of technical tools very much used in Digital, like Google Analytics (Web Analytics) or NetBase (for Social Media). For example, I’ve had to take the Google Analytics Academy courses in order to get the certificates I need to be confident with the tool.

I do not consider myself an expert in Google Analytics (GA) yet, but now I am the person in charge of it for all the accounts and properties we have. As such, I’ve had to give some trainings in GA, with a very special focus in E-Commerce and Attribution Modeling. And I have received a positive feedback, so now I am much more confident about my GA knowledge and skills.

### Next Steps

There are some other tasks and projects I’ve been involved in, but I think the points given are enough to give you a glimpse of what I’ve been able to do in Digital, without generating any confidentiality issue.

Due to COVID-19, this has been a very accelerated journey, with tons of challenges and learnings that have made me a more knowledgeable and skilled person every single day. So much that I firmly believe I am now in many ways better to whom I was last month.

I know this journey is not over, and that I will be facing many challenges. But I will always embrace every challenge, completely committed to my personal development and to keep growing in this new area, Digital. I’ll try to keep you posted about this journey.

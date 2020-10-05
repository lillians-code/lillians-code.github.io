---
title: "Seek.com job analysis"
excerpt: "A thorough analysis of data jobs using information I web scraped from Seek.com"
date: 2020-09-23T18:38:52+00:00
header:
  image: /assets/images/seek/seek.png
  teaser: assets/images/seek/seek.png
# gallery:
#   - url: /assets/images/pos.jpg
#     image_path: assets/images/pos.jpg
#     alt: "Positive sentiment"
#   - url: /assets/images/neu.jpg
#     image_path: assets/images/neu.jpg
#     alt: "Neutral sentiment"
#   - url: /assets/images/neg.jpg
#     image_path: assets/images/neg.jpg
#     alt: "Negative sentiment"
---
[See Source Code](https://github.com/cyan-sunset/seek.com-data-jobs){: .btn .btn--info}

For this project I wanted to answer two main questions.

1. Which factors impact salary the most for data related roles?
2. What are the main differences between data related jobs in Sydney and Melbourne?

### Step 1: Web scraping

I scraped seek.com.au using BeautifulSoup for information that I needed to answer my questions. 

I chose to scrape seek.com.au rather than another job aggregator because it was the only popular job aggregator website that had the option to search using ranges (eg. $50,000-$60,000) rather than a minimum salary limit (ie. $50,000+). Most job listings don't have a clearly defined, per annum salary, so the idea was to get a rough estimate using the salary range search option (eg. job A doesn't have a salary but showed up in the search for $50-$60k, so I could use either $50k, $60k, or the average $55k for analyses). But, after I actually scraped the data I realised that the same listing would often show up for multiple salary range searches, so I ditched that idea and decided to use only the listings I had real salaries for. 

This meant that I had a lot less data than initially anticipated, so I expanded my list of search terms in hopes of getting more data-related job listings for a more legitimate analysis, which seems to have worked.

To find listing for data jobs, I searched the website (using urls, and the requests package) using the following searchterms:

- Data scientist
- Data engineer
- Data analyst
- Data warehousing
- Big data
- Business analyst
- Data

This is an example of a job listing on seek.com.au. This is the format of job listings on the search page, which has 20 job listing results per page with the exception of time when there are "promoted" job listings, which will be at the top of the page and are additional to the base 20 results. 

![example job]({{ site.url }}{{ site.baseurl }}/assets/images/seek/example-job.png){: .align-center}

As you can see, there is already a lot of data to be scraped from this search result. Scraping information from one page (ie. the search page with many listings) is much more efficient than scraping information from the full page post of each job listing because I only have to request the search page to get information about 20 jobs instead of requesting 20 different pages to get information about the same 20 jobs. That being said, to cover my bases I did also scrape the information from each full posting because I noticed that the salary information would sometimes not be available in the search result, but would appear in the full posting, also I wanted the full description of each job to have more data for my NLP analysis later on.

However, some job listings showed up in multiple different searches, so I had to have some way of removing duplicate job listings for a more accurate analysis. I did this by breaking up the web scraping in two different stages - first scraping information that I could use to drop duplicates (ie. job title, job link, company name, short description, job location, job area), dropping duplicates using these fields to get unique job listings, and scraping the remaining information I needed only from the unique jobs. Of course, I could've scraped all the information I wanted at once and dropped duplicates afterwards, but it was much more time efficient to do it this way because:

1. All that information was readily available on the search page and
2. A majority of listings were actually duplicates so it would have unnecessarily taken much longer to accomplish the same thing.

Before dropping duplicates, I had 79,322 jobs in my dataframe, and after dropping duplicates I had only 17,766. 

All in all, I managed to scrape the following data for 17,766 jobs:

- Job title
- Job link (url link to job posting so I can revisit if needed)
- Company name
- Date posted
- Location (Melbourne, Sydney, etc.)
- Area (Suburb or more specific location)
- Job category/industry
- Work type (Full/part time, casual, contract)
- Salary (from search result and full posting)
- Short description from search result
- Long description from full posting

### Step 2: Data cleaning

There was some cleaning to be done for the salaries and job descriptions of this data.

#### Salary

The salary data came as a real mixed bunch. A majority of the job listings did not include anything for the salary field. Out of the listings that had information in the salary field, some of them were the rates per hour, some were project rates, and some were not even numbers (employer specified alternate remuneration). Even when the job listing did have salary per annum (which is what I wanted), the salary often came as a range (eg. $53,646.87 - $55,828.98) rather than a single value. Sometimes, the salary would be specified as "$50k", or "$50k-$60k". 

My solution for the salaries was to drop NA values, and use regex to select only the numerical values that were 5 digits or more (most likely to be indicative of an annual salary rather than per hour or per project). I also used regex to remove the "k"s in the salaries before multiplying the resulting number by 1000 to get an actual numerical value to work with.

#### Job descriptions

I needed to remove all punctuation, html tags, and newline markers (\n) from the job descriptions. To do this, I defined a custom function, and applied it to all the raw descriptions.

### Step 3: Modelling

#### Question 1 - Factors that impact salary

I investigated which words in job titles and job descriptions that best predict job salary.

I framed these questions as classification problems by dividing job salaries into high and low groups using the median salary. 

![example job]({{ site.url }}{{ site.baseurl }}/assets/images/seek/job-titles-highlow.png){: .align-center}

![example job]({{ site.url }}{{ site.baseurl }}/assets/images/seek/job-desc-highlow.png){: .align-center}

<!-- {% include gallery caption="Emotions." %} -->

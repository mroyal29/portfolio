# Sentiment Asymmetry in Social Media: How Negative Political Content Drives Disproportionate Engagement

This repository contains the final project for UC Berkeley Masters in Data Scince (MIDS) Statistics for Data Science class (W203). The goal of this project was to build a clear, interpretable descriptive model to understand how tweet sentiment and topic relate to social media engagement. Model simplicity and transparency were prioritized to support meaningful interpretation of user behavior and platform incentives. We leveraged RoBERTa sentiment and topic natural language processing (NLP) models to create the features necessary for our model. 

**Project Overview**
* Dataset: 115k English-language tweets from March 2023
* Source: Hugging Face Twitter 100M [tweets](https://huggingface.co/datasets/enryu43/twitter100m_tweets) & [users](https://huggingface.co/datasets/enryu43/twitter100m_users)
* Engagement Metric: Aggregate of likes, retweets, replies, and quotes
* Approach: NLP-driven feature engineering + linear regression with minimal controls

**Methodology: Sampling & Filtering** 
* Reservoir sampling from 100M+ tweets to enable efficient, unbiased selection
* Restricted to March 2023 to align tweet timing with follower counts
* Filtered out:
    * Non-English tweets
    * Replies and zero-engagement tweets
    * Tweets containing links or videos
    * Users with <100 followers or in the top 5% by follower count
    * Extremely short (1–2 words) or long (>75 words) tweets
* Final analytic dataset: 115,258 tweets

**Feature Engineering & NLP**
* Language Detection
        * langdetect to retain only English-language tweets
* Topic Classification
        * [Twitter-specific RoBERTA model for topic classification](https://huggingface.co/cardiffnlp/tweet-topic-base-multilingual)
        * Tweets classified as:
                * Politics (News & Social Concern)
                * Other
* Sentiment Analysis
        * [Twitter-specific RoBERTa model for sentiment classification](https://huggingface.co/cardiffnlp/twitter-roberta-base-sentiment-latest)

**Modeling Approach:**
* Model Type: Ordinary Least Squares (OLS) regression
* Outcome Variable: Log-transformed engagement score
* Controls:
           * Follower count
           * Word count
* Rationale:
        * Linear models allow direct interpretation of sentiment–engagement relationships
        * Focused on understanding direction, magnitude, and differences across sentiment-topic combinations

**Key Findings**
* Negative political tweets generate approximately 176 more engagement than neutral political tweets
* Positive political sentiment shows negligible engagement gains
* Non-political content exhibits minimal engagement variation across sentiment levels
* Results suggest social media ecosystems may structurally amplify negative political sentiment

**Project Contributors**<br>
* Mannan Mishra: mannan_mishra@berkeley.edu
* Maagie Royal: mroyal7@.berkeley.edu,
* Lyn Wang: fulingw@ischool.berkeley.edu
* Ale Alvarez: alexalva@ischool.berkeley.edu <br><br>

**Final Report:** [link](https://github.com/mroyal29/portfolio/blob/main/Sentiment%20Asymmetry%20in%20Social%20Media/Report.pdf)<br><br>

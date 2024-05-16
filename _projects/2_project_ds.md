---
layout: page
title: Real Time Stock Market Data Engineering
description: Build the archetecture for Streaming Real Time Stock Market Data using Kafka and AWS.
img: assets/img/real-time-stock-market-data-streaming-pic1.jpg
importance: 2
category: Data Science
related_publications:
---

The project is End-To-End Data Engineering Project on Real-Time Stock Market Data using the power of Kafka and AWS. The project aims to provide real-time stock market data to the end-users. The project is divided into three parts:
1. Data Collection
    * Utilize the Alpaca API to collect real-time stock market data. At this moment, we only use free plan to collect data.
2. Data Processing
    * Utilize Kafka Producer and Kafka Producer and Consumer on top of AWS EC2.
    * (The Kafka Producer API allows applications to send streams of data to the Kafka cluster. The Kafka Consumer API allows applications to read streams of data from the cluster)
3. Data Streaming
    * Utilize AWS S3, Crawler, AWS Glue Data Catalog, and Athena to store and query the data.

<a href="https://github.com/idsts2670/real-time-stock-market-data-streaming">Github Code Link</a>


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/real-time-stock-market-data-streaming-pic1.jpg" title="Global Warming" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

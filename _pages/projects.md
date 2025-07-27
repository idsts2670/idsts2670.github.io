---
layout: page
title: projects
permalink: /projects/
description: A comprehensive showcase of my data science, quantitative finance, and software development projects
nav: true
nav_order: 2
display_categories: [Quantitative Finance, Data Science, Visualization]
horizontal: false
---

## Project Highlights

<div class="project-highlights">
  <div class="highlight-card">
    <h3>🔢 Quantitative Trading Systems</h3>
    <p>Advanced algorithmic trading strategies using statistical models, options pricing, and real-time market data analysis. Built with Python, Go, and various financial APIs.</p>
    <a href="https://github.com/idsts2670/quant_algo_trading" class="project-link">View Repository →</a>
  </div>
  
  <div class="highlight-card">
    <h3>🤖 Alpaca MCP Server</h3>
    <p>Model Context Protocol server for Alpaca trading platform, enabling seamless integration between AI models and financial markets for automated trading decisions.</p>
    <a href="https://github.com/idsts2670/alpaca-mcp-server" class="project-link">View Repository →</a>
  </div>
  
  <div class="highlight-card">
    <h3>📊 Real-Time Data Streaming</h3>
    <p>Engineering real-time stock market data streaming systems using Apache Kafka and AWS services for high-frequency trading analytics.</p>
    <a href="https://github.com/idsts2670/real-time-stock-market-data-streaming" class="project-link">View Repository →</a>
  </div>
</div>

<!-- pages/projects.md -->
<div class="projects">
{%- if site.enable_project_categories and page.display_categories %}
  <!-- Display categorized projects -->
  {%- for category in page.display_categories %}
  <h2 class="category">{{ category }}</h2>
  {%- assign categorized_projects = site.projects | where: "category", category -%}
  {%- assign sorted_projects = categorized_projects | sort: "importance" %}
  <!-- Generate cards for each project -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-2">
    {%- for project in sorted_projects -%}
      {% include projects_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for project in sorted_projects -%}
      {% include projects.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
  {% endfor %}

{%- else -%}
<!-- Display projects without categories -->
  {%- assign sorted_projects = site.projects | sort: "importance" -%}
  <!-- Generate cards for each project -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-2">
    {%- for project in sorted_projects -%}
      {% include projects_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for project in sorted_projects -%}
      {% include projects.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
{%- endif -%}
</div>

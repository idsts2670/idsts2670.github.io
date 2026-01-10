---
layout: about
title: about
permalink: /
subtitle: 

profile:
  align: right
  image: prof_pic.jpg
  image_circular: false # crops the image to make it circular
  
  more_info: >
      <p>(765)-476-3830</p>
      <p>idsts2670@gmail.com</p>
      <p>Located in IL, USA</p>

news: false  # includes a list of news items
latest_posts: false  # includes a list of the newest posts
selected_papers: false # includes a list of papers marked as "selected={true}"
social: true  # includes social icons at the bottom of the page
---

<div class="hero-section">
  <h1 class="hero-title">
    Hi, <span class="hero-accent">Welcome</span>
  </h1>
  
  <div class="hero-subtitle">
    I'm Satoshi Ido,<br>
    Technical Marketing | Algorithmic Trading | Statistical Analysis
  </div>
  
  <div class="hero-description">
    Working at the intersection of algorithmic trading, data science, and technical storytelling.
  </div>
</div>

## GitHub Activity

<div class="github-stats">
  <div class="stats-grid">
    <div class="stat-card">
      <img src="https://github-readme-stats-idsts2670.vercel.app/api?username=idsts2670&show_icons=true&theme=default&hide_border=true&bg_color=ffffff&title_color=1d1e20&text_color=1d1e20&icon_color=667eea" alt="GitHub Stats" />
    </div>
    <div class="stat-card">
      <img src="https://github-readme-stats-idsts2670.vercel.app/api/top-langs/?username=idsts2670&layout=compact&theme=default&hide_border=true&bg_color=ffffff&title_color=1d1e20&text_color=1d1e20" alt="Top Languages" />
    </div>
  </div>
</div>

## Featured Projects

<div class="project-intro">
  <p>A showcase of my current GitHub pinned repositories, spanning quantitative finance, data science, and software development.</p>
</div>

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% for repo in site.data.repositories.github_repos limit:6 %}
    {% include repository/repo.html repository=repo %}
  {% endfor %}
</div>

<div class="repo-actions">
  <a href="{{ '/repositories/' | relative_url }}" class="btn-repo-more">
    View All Projects →
  </a>
</div>
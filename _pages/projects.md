---
layout: page
title: Projects
permalink: /projects/
# description: A growing collection of your cool projects.
nav: true
nav_order: 3
display_categories: [research, engineering, fun]
horizontal: false
---

My current research is on improving and better evaluating Faithfulness and Factuality of Language Models, focusing on data-driven and modeling methodologies. I also want to work on combatting misinformation. Other research experiences include [Vision-language in Robotics](https://github.com/hieuchi911/LLaMA-Factory-Sober) and Deep learning system testing at [DeepUSC Research](https://deep-usc.org/people), etc. A full list of past projects can be found in my [resume](https://hieuchi911.github.io/assets/pdf/my_resume.pdf).

Beyond my research and engineering projects, I enjoy covering songs and making music videos.

<!-- pages/projects.md -->
<div class="projects">
{% if site.enable_project_categories and page.display_categories %}
  <!-- Display categorized projects -->
  {% for category in page.display_categories %}
  <a id="{{ category }}" href=".#{{ category }}">
    <h2 class="category" style="color: #828282"> {{ category }}</h2>
  </a>
  {% assign categorized_projects = site.projects | where: "category", category %}
  {% assign sorted_projects = categorized_projects | sort: "importance" %}
  <!-- Generate cards for each project -->
  {% if page.horizontal %}
  <div class="container">
    <div class="row row-cols-1 row-cols-md-2">
    {% for project in sorted_projects %}
      {% include projects_horizontal.liquid %}
    {% endfor %}
    </div>
  </div>
  {% else %}
  <div class="row row-cols-1 row-cols-md-3">
    {% for project in sorted_projects %}
      {% include projects.liquid %}
    {% endfor %}
  </div>
  {% endif %}
  {% endfor %}

{% else %}

<!-- Display projects without categories -->

{% assign sorted_projects = site.projects | sort: "importance" %}

  <!-- Generate cards for each project -->

{% if page.horizontal %}

  <div class="container">
    <div class="row row-cols-1 row-cols-md-2">
    {% for project in sorted_projects %}
      {% include projects_horizontal.liquid %}
    {% endfor %}
    </div>
  </div>
  {% else %}
  <div class="row row-cols-1 row-cols-md-3">
    {% for project in sorted_projects %}
      {% include projects.liquid %}
    {% endfor %}
  </div>
  {% endif %}
{% endif %}
</div>

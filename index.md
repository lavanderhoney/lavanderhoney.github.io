---
# You don't need to edit this file, it's empty on purpose.
# Edit theme's home layout instead if you wanna make some changes
# See: https://jekyllrb.com/docs/themes/#overriding-theme-defaults
layout: single
author_profile: true
---

I'm Milap, an undergraduate CSE student passionate about Machine Learning, Computer Vision, and AI-driven innovation.  
I enjoy working on real-world datasets and building end-to-end AI solutions.

## 📂 Projects

<div class="projects-grid">
{% for project in site.projects %}
  <div class="project-card">
    <a href="{{ project.url | relative_url }}">
      {% if project.image %}
        <img src="{{ project.image | relative_url }}" alt="{{ project.title }}" style="max-width:100%; height:auto; display:block;">
      {% elsif project.header and project.header.overlay_image %}
        <img src="{{ project.header.overlay_image | relative_url }}" alt="{{ project.title }}" style="max-width:100%; height:auto; display:block;">
      {% endif %}
      <h3 style="margin-top:0.4rem">{{ project.title }}</h3>
    </a>
    {% if project.excerpt %}<p>{{ project.excerpt }}</p>{% endif %}
  </div>
{% endfor %}
</div>

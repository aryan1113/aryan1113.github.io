---
last_modified: 2025-06-26 09:40:49 +0530
layout: default
published: true
title: Work Experience
permalink: /workex/
---

## Work Experiences
{% for job in site.data.experiences.work %}
### {{ job.company }}, {{ job.location }}

{% for role in job.roles %}
**Role:** {{ role.title }}  
**Duration:** {{ role.start_date | date: "%b %Y" }}{% if role.current %} - Present{% elsif role.end_date %} - {{ role.end_date | date: "%b %Y" }}{% endif %}  
**Type:** {{ role.type }}  
**Process:** {{ role.process }}

**What I Learned:**
{% for learning in role.learnings %}
- {{ learning }}
{% endfor %}

{% if role.projects.size > 0 %}
**Projects:**
{% for project in role.projects %}
- {{ project }}
{% endfor %}
{% endif %}

---
{% endfor %}
{% endfor %}


## Talks & Events Attended

{% for talk in site.data.experiences.talks %}
### {{ talk.event }}
**Date:** {% if talk.date_from == talk.date_to %}  {{ talk.date_from | date: "%d %B %Y" }}
{% else %}  {{ talk.date_from | date: "%d %B %Y" }} - {{ talk.date_to | date: "%d %B %Y" }}
{% endif %}  
**Type:** {{ talk.type }}  

**Key Takeaways:**
{% for takeaway in talk.takeaways %}
- {{ takeaway }}
{% endfor %}

{% endfor %}

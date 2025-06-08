---
layout: default
published: true
title: Work Experience
permalink: /workex/
---

## Work Experiences
{% for job in site.data.experiences.work %}
### {{ job.company }}, {{ job.location }}
**Role:** {{ job.role }} \
**Duration:** {{ job.start_date | date: "%b %Y" }}{% if job.current %} - Present{% elsif job.end_date %} - {{ job.end_date | date: "%b %Y" }}{% endif %}  \
**Type:** {{ job.type }} \
**Process:** {{ job.process }}

**What I Learned:**
{% for learning in job.learnings %}
- {{ learning }}
{% endfor %}

{% if job.projects.size > 0 %}
**Projects:**
{% for project in job.projects %}
- {{ project }}
{% endfor %}
{% endif %}

---
{% endfor %}

## Talks & Events Attended

{% for talk in site.data.experiences.talks %}
### {{ talk.event }}
**Date:** {{ talk.date_from | date: "%d %B %Y" }}  - {{talk.date_to}}\
**Type:** {{ talk.type }}  

**Key Takeaways:**
{% for takeaway in talk.takeaways %}
- {{ takeaway }}
{% endfor %}

{% endfor %}
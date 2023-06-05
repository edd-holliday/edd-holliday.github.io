---
layout: page
title: Resume
---

## Education

{% for education in site.data.resume.education %}
### {{ education.degree }}
**{{ education.university }}** - {{ education.city }}
_{{ education.duration }}_
{% endfor %}

## Work Experience

{% for experience in site.data.resume.work_experience %}
### {{ experience.job_title }}
**{{ experience.company }}** - {{ experience.city }}
_{{ experience.duration }}_
{% endfor %}


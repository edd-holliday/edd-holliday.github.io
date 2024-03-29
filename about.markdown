---
layout: page
title: About Me
permalink: /about/
---

## Introduction

Hello and welcome! My name is Edd Holliday, an accomplished professional with a rich tapestry of experience spanning technology, ESG, Aerospace, Defence, and Finance. My educational journey began with a BSc in Computer Science from Durham University, followed by a Postgraduate Diploma in International Rural Development and a Level 5 Certificate in Management & Leadership from the Chartered Management Institute. I am currently strudying for my Investment Managers Certificate.

My career has been equally diverse. A decade in the British Army offered me unparalleled experiences in leadership, management, and cooperation with global teams. Solving Complex problems in challegning environments. This foundation has enabled me to excel in all my professional pursuits, including my recent role in the Defence and Aerospace sector, where I leveraged my educational background and analytical acumen to navigate complex market dynamics and emerging technologies.

Due to my diverse experiences, I consider myself more of a generalist than a specialist. I thrive on understanding the interconnections between different fields and continuously seek to learn and grow. A ssytems thinker. This passion for lifelong learning has allowed me to excel in various roles, from strategic planning and policy analysis to technical due diligence and M&A.

Thank you for visiting, and feel free to reach out for a copy of my resume.


## Professional
<ul>
{% for role in site.data.resume %}
    <li>
        <h3> {{ role.job_title }} </h3>
        <h4> {{ role.company }} - {{ role.duration }} </h4>
        {{ role.achievements }}
    </li>
{% endfor %}
</ul>

## Education
<ul>
{% for education in site.data.education %}
    <li>
        <h3>{{ education.degree }}</h3>
        <h4>{{ education.university }} - {{ education.duration }} </h4>
    </li>
{% endfor %}
</ul>
---
layout: page
title: Statistics
permalink: /statistics
---

<table class="minimalistBlack">
  <thead>
  <tr>
    <th>Post</th>
    <th>Date</th>
    <th>Hits (From April 14, 2020)</th>
  </tr>
  </thead>
  <tbody>
  {% for post in site.posts %}
    <tr>
      <td>{{ post.title }} </td>
      <td>{{ post.date | date: '%B %Y' }} </td>
<td>
{% if site.url contains 'localhost' %}

{% else %}
<img src="https://hits.seeyoufarm.com/api/count/incr/badge.svg?url={{ site.url }}{{ page.url }}&count_bg=%23555555&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false"/>
{% endif %}
</td>
    </tr>
  {% endfor %}
  </tbody>
</table>
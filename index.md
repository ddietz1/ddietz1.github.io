---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
title: home
---
# projects
{% for project in site.data.projects %}
- **{{ project.title }}**
{% endfor %}

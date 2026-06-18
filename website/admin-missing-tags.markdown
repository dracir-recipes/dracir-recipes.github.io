---
layout: page
title: Admin - Missing Recipe Tags
permalink: /admin/missing-tags/
---

# Missing Recipe Tags

This page lists tags used in recipe front matter (`recipe-tags`) that are not defined in `_data/recipe-tags.yml`.

{% assign configured_tags_blob = "" %}
{% for group in site.data.recipe-tags.groups %}
  {% for tag in group.tags %}
    {% assign configured_label = tag.label | strip | downcase %}
    {% if configured_label != "" %}
      {% assign configured_tags_blob = configured_tags_blob | append: configured_label | append: "||" %}
    {% endif %}
  {% endfor %}
{% endfor %}
{% assign configured_tags = configured_tags_blob | split: "||" | uniq %}

{% assign used_tags_blob = "" %}
{% for recipe in site.recipes %}
  {% for raw_tag in recipe.recipe-tags %}
    {% assign normalized_tag = raw_tag | strip | downcase %}
    {% if normalized_tag != "" %}
      {% assign used_tags_blob = used_tags_blob | append: normalized_tag | append: "||" %}
    {% endif %}
  {% endfor %}
{% endfor %}
{% assign used_tags = used_tags_blob | split: "||" | uniq | sort %}

{% assign missing_tags_blob = "" %}
{% for used_tag in used_tags %}
  {% if used_tag != "" %}
    {% unless configured_tags contains used_tag %}
      {% assign missing_tags_blob = missing_tags_blob | append: used_tag | append: "||" %}
    {% endunless %}
  {% endif %}
{% endfor %}
{% assign missing_tags = missing_tags_blob | split: "||" | uniq | sort %}
{% assign missing_count = 0 %}
{% for missing_tag in missing_tags %}
  {% if missing_tag != "" %}
    {% assign missing_count = missing_count | plus: 1 %}
  {% endif %}
{% endfor %}

{% assign unused_tags_blob = "" %}
{% for configured_tag in configured_tags %}
  {% if configured_tag != "" %}
    {% unless used_tags contains configured_tag %}
      {% assign unused_tags_blob = unused_tags_blob | append: configured_tag | append: "||" %}
    {% endunless %}
  {% endif %}
{% endfor %}
{% assign unused_tags = unused_tags_blob | split: "||" | uniq | sort %}
{% assign unused_count = 0 %}
{% for unused_tag in unused_tags %}
  {% if unused_tag != "" %}
    {% assign unused_count = unused_count | plus: 1 %}
  {% endif %}
{% endfor %}

{% if missing_count > 0 %}
<p class="text-muted">{{ missing_count }} missing tag{% if missing_count != 1 %}s{% endif %} found.</p>

<div class="list-group mb-4">
  {% for missing_tag in missing_tags %}
    {% if missing_tag != "" %}
    <div class="list-group-item">
      <h2 class="h6 mb-2">{{ missing_tag }}</h2>
      <ul class="mb-0">
        {% for recipe in site.recipes %}
          {% assign has_missing_tag = false %}
          {% for raw_tag in recipe.recipe-tags %}
            {% assign normalized_tag = raw_tag | strip | downcase %}
            {% if normalized_tag == missing_tag %}
              {% assign has_missing_tag = true %}
            {% endif %}
          {% endfor %}
          {% if has_missing_tag %}
            <li><a href="{{ recipe.url | relative_url }}">{{ recipe.title }}</a></li>
          {% endif %}
        {% endfor %}
      </ul>
    </div>
    {% endif %}
  {% endfor %}
</div>
{% else %}
<p class="alert alert-success">All tags used in recipes are defined in <code>_data/recipe-tags.yml</code>.</p>
{% endif %}

<hr class="my-4">

<h2 class="h4">Configured Tags Not Used by Any Recipe</h2>

{% if unused_count > 0 %}
<p class="text-muted">{{ unused_count }} configured tag{% if unused_count != 1 %}s{% endif %} currently unused.</p>
<ul class="list-group mb-4">
  {% for unused_tag in unused_tags %}
    {% if unused_tag != "" %}
    <li class="list-group-item">{{ unused_tag }}</li>
    {% endif %}
  {% endfor %}
</ul>
{% else %}
<p class="alert alert-success">Every configured tag in <code>_data/recipe-tags.yml</code> is used by at least one recipe.</p>
{% endif %}

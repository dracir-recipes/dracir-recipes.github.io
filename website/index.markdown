---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
---

<h1>Latest Recipes</h1>

<div class="row mb-3 g-3 text-center">
{% for recipe in site.recipes %}
<div class="col-sm-4">
<div class="col">
	<div class="card">
		<a href="{{ recipe.url }}">
			<img src="..." class="card-img-top" alt="...">
		</a>
		<div class="card-body">
			<h5 class="card-title">{{ recipe.title }}</h5>
			<p class="card-text"></p>
		</div>
	</div>
	</div>
</div>
{% endfor %}
</div>

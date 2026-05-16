---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
---

<h1>Cookbook</h1>

<h2> filters </h2>
<ul>
	<li><span class="badge rounded-pill text-bg-primary">Breakfast</span></li>
	<li><span class="badge rounded-pill text-bg-secondary">Meat</span></li>
	<li><span class="badge rounded-pill text-bg-success">Vegetarian</span></li>
	<li><span class="badge rounded-pill text-bg-warning">Snacks</span></li>
</ul>

<h2> recipes </h2>

<div class="row mb-3 g-3 text-center recipes-div">
{% for recipe in site.recipes %}
<div class="col-sm-4">
<div class="col">
	<a href="{{ recipe.url }}">
		<div class="card">
			<div class="card-body">
			<img src="{{ recipe.thumbnail }}" class="card-img-top" alt="{{ recipe.title }}">
				<h5 class="card-title">{{ recipe.title }}</h5>
			</div>
		</div>
	</a>
	</div>
</div>
{% endfor %}
</div>

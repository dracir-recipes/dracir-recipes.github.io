---
layout: page
title: Admin - Recipe Status Dashboard
permalink: /admin/status-dashboard/
---

<style>
	.status-dashboard {
		--bg-0: #f8fafc;
		--bg-1: #e2e8f0;
		--ink: #0f172a;
		--muted: #475569;
		--danger: #dc2626;
		--warning: #f59e0b;
		--progress: #f97316;
		--done: #16a34a;
		--card: #ffffff;
		--ring: rgba(15, 23, 42, 0.08);
		background:
			radial-gradient(circle at 10% 0%, rgba(249, 115, 22, 0.14), transparent 45%),
			radial-gradient(circle at 90% 20%, rgba(22, 163, 74, 0.14), transparent 40%),
			linear-gradient(180deg, var(--bg-0) 0%, var(--bg-1) 100%);
		border-radius: 20px;
		padding: 1.5rem;
		color: var(--ink);
	}

	.status-header h1 {
		font-size: clamp(1.6rem, 2.5vw, 2.2rem);
		font-weight: 800;
		letter-spacing: 0.02em;
		margin-bottom: 0.35rem;
	}

	.status-header p {
		margin: 0;
		color: var(--muted);
	}

	.kpi-grid {
		display: grid;
		grid-template-columns: repeat(4, minmax(0, 1fr));
		gap: 0.8rem;
		margin-top: 1rem;
	}

	.kpi-card {
		background: var(--card);
		border: 1px solid var(--ring);
		border-radius: 14px;
		padding: 0.85rem;
		box-shadow: 0 4px 20px rgba(2, 6, 23, 0.05);
	}

	.kpi-label {
		font-size: 0.8rem;
		color: var(--muted);
		text-transform: uppercase;
		letter-spacing: 0.05em;
		margin-bottom: 0.3rem;
	}

	.kpi-value {
		font-size: 1.7rem;
		font-weight: 800;
		line-height: 1;
	}

	.progress-wrap {
		margin-top: 1rem;
		background: var(--card);
		border: 1px solid var(--ring);
		border-radius: 14px;
		padding: 0.85rem;
	}

	.progress-label {
		display: flex;
		justify-content: space-between;
		font-weight: 600;
		margin-bottom: 0.45rem;
	}

	.progress {
		height: 14px;
		background: #dbe4ef;
		border-radius: 999px;
	}

	.progress-bar {
		font-weight: 700;
	}

	.status-groups {
		display: grid;
		grid-template-columns: repeat(2, minmax(0, 1fr));
		gap: 0.9rem;
		margin-top: 1rem;
	}

	.status-panel {
		background: var(--card);
		border: 1px solid var(--ring);
		border-radius: 14px;
		padding: 0.9rem;
		box-shadow: 0 4px 20px rgba(2, 6, 23, 0.05);
	}

	.status-panel h2 {
		font-size: 1rem;
		font-weight: 800;
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 0.5rem;
	}

	.status-badge {
		font-size: 0.78rem;
		font-weight: 700;
		padding: 0.22rem 0.55rem;
		border-radius: 999px;
		background: #f1f5f9;
		border: 1px solid #dbe4ef;
	}

	.status-list {
		margin: 0;
		padding-left: 1.1rem;
		max-height: 340px;
		overflow: auto;
	}

	.status-list li {
		margin-bottom: 0.2rem;
	}

	.status-list a {
		text-decoration: none;
	}

	.status-list a:hover {
		text-decoration: underline;
	}

	.status-danger h2 {
		color: var(--danger);
	}

	.status-warning h2 {
		color: var(--warning);
	}

	.status-progress h2 {
		color: var(--progress);
	}

	.status-done h2 {
		color: var(--done);
	}

	@media (max-width: 992px) {
		.kpi-grid {
			grid-template-columns: repeat(2, minmax(0, 1fr));
		}
		.status-groups {
			grid-template-columns: 1fr;
		}
	}

	@media (max-width: 576px) {
		.status-dashboard {
			padding: 1rem;
			border-radius: 14px;
		}
		.kpi-value {
			font-size: 1.45rem;
		}
	}
</style>

{% assign recipe_total = site.recipes.size %}

{% assign blocked_blob = "" %}
{% assign todo_blob = "" %}
{% assign inprogress_blob = "" %}
{% assign done_blob = "" %}

{% for recipe in site.recipes %}
	{% assign raw_status = recipe.status | default: "" | strip %}
	{% assign normalized = raw_status | replace: "️", "" %}

	{% if normalized == "🟥" %}
		{% assign blocked_blob = blocked_blob | append: recipe.title | append: "||" %}
	{% elsif normalized == "🟨" %}
		{% assign todo_blob = todo_blob | append: recipe.title | append: "||" %}
	{% elsif normalized == "🟧" %}
		{% assign inprogress_blob = inprogress_blob | append: recipe.title | append: "||" %}
	{% elsif normalized == "🟩" %}
		{% assign done_blob = done_blob | append: recipe.title | append: "||" %}
	{% else %}
		{% assign todo_blob = todo_blob | append: recipe.title | append: "||" %}
	{% endif %}
{% endfor %}

{% assign blocked_list = blocked_blob | split: "||" | uniq | sort %}
{% assign todo_list = todo_blob | split: "||" | uniq | sort %}
{% assign inprogress_list = inprogress_blob | split: "||" | uniq | sort %}
{% assign done_list = done_blob | split: "||" | uniq | sort %}

{% assign blocked_count = 0 %}
{% for entry in blocked_list %}{% if entry != "" %}{% assign blocked_count = blocked_count | plus: 1 %}{% endif %}{% endfor %}
{% assign todo_count = 0 %}
{% for entry in todo_list %}{% if entry != "" %}{% assign todo_count = todo_count | plus: 1 %}{% endif %}{% endfor %}
{% assign inprogress_count = 0 %}
{% for entry in inprogress_list %}{% if entry != "" %}{% assign inprogress_count = inprogress_count | plus: 1 %}{% endif %}{% endfor %}
{% assign done_count = 0 %}
{% for entry in done_list %}{% if entry != "" %}{% assign done_count = done_count | plus: 1 %}{% endif %}{% endfor %}

{% assign percent = 0 %}
{% if recipe_total > 0 %}
	{% assign percent = done_count | times: 100 | divided_by: recipe_total %}
{% endif %}

<section class="status-dashboard">
	<div class="status-header">
		<h1>Recipe Status Dashboard</h1>
		<p>Track recipe completion and quickly jump to entries by status.</p>
	</div>

	<div class="kpi-grid" role="list" aria-label="Status summary">
		<div class="kpi-card" role="listitem">
			<div class="kpi-label">Total Recipes</div>
			<div class="kpi-value">{{ recipe_total }}</div>
		</div>
		<div class="kpi-card" role="listitem">
			<div class="kpi-label">Completed 🟩</div>
			<div class="kpi-value">{{ done_count }}</div>
		</div>
		<div class="kpi-card" role="listitem">
			<div class="kpi-label">In Progress 🟧</div>
			<div class="kpi-value">{{ inprogress_count }}</div>
		</div>
		<div class="kpi-card" role="listitem">
			<div class="kpi-label">Needs Work 🟨 + 🟥</div>
			<div class="kpi-value">{{ todo_count | plus: blocked_count }}</div>
		</div>
	</div>

	<div class="progress-wrap">
		<div class="progress-label">
			<span>Completion Progress</span>
			<span>{{ percent }}%</span>
		</div>
		<div class="progress" role="progressbar" aria-label="Recipe completion" aria-valuenow="{{ percent }}" aria-valuemin="0" aria-valuemax="100">
			<div class="progress-bar bg-success" style="width: {{ percent }}%;">{{ percent }}%</div>
		</div>
	</div>

	<div class="status-groups">
		<section class="status-panel status-danger">
			<h2>Blocked / Red <span class="status-badge">{{ blocked_count }}</span></h2>
			<ol class="status-list">
				{% for title in blocked_list %}
					{% if title != "" %}
						{% assign recipe = site.recipes | where: "title", title | first %}
						{% if recipe %}
							<li><a href="{{ recipe.url | relative_url }}">{{ recipe.title }}</a></li>
						{% endif %}
					{% endif %}
				{% endfor %}
			</ol>
		</section>

		<section class="status-panel status-warning">
			<h2>To Do / Yellow <span class="status-badge">{{ todo_count }}</span></h2>
			<ol class="status-list">
				{% for title in todo_list %}
					{% if title != "" %}
						{% assign recipe = site.recipes | where: "title", title | first %}
						{% if recipe %}
							<li><a href="{{ recipe.url | relative_url }}">{{ recipe.title }}</a></li>
						{% endif %}
					{% endif %}
				{% endfor %}
			</ol>
		</section>

		<section class="status-panel status-progress">
			<h2>In Progress / Orange <span class="status-badge">{{ inprogress_count }}</span></h2>
			<ol class="status-list">
				{% for title in inprogress_list %}
					{% if title != "" %}
						{% assign recipe = site.recipes | where: "title", title | first %}
						{% if recipe %}
							<li><a href="{{ recipe.url | relative_url }}">{{ recipe.title }}</a></li>
						{% endif %}
					{% endif %}
				{% endfor %}
			</ol>
		</section>

		<section class="status-panel status-done">
			<h2>Done / Green <span class="status-badge">{{ done_count }}</span></h2>
			<ol class="status-list">
				{% for title in done_list %}
					{% if title != "" %}
						{% assign recipe = site.recipes | where: "title", title | first %}
						{% if recipe %}
							<li><a href="{{ recipe.url | relative_url }}">{{ recipe.title }}</a></li>
						{% endif %}
					{% endif %}
				{% endfor %}
			</ol>
		</section>
	</div>
</section>

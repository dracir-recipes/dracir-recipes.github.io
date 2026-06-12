---
layout: page
title: Recipe Swipe Picker
permalink: /swipe/
---

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Source+Sans+3:wght@400;600;700&display=swap" rel="stylesheet">

{% assign recipes = site.recipes | sort: "title" %}
<section class="swipe-lab" data-swipe-root>
	<header class="swipe-head">
		<div>
			<a href="{{ '/' | relative_url }}" class="btn btn-outline-secondary btn-sm swipe-back-link">Back to cookbook</a>
			<h1 class="swipe-title">Pick Tonight's Recipe</h1>
			<p class="swipe-subtitle">Swipe left to skip, swipe right to shortlist. Use touch, mouse drag, or keyboard arrows.</p>
		</div>
	</header>

	<div class="swipe-layout">
		<div class="swipe-stage">
			<div class="mb-3">
				<label for="swipe-type-filter" class="form-label fw-semibold mb-1">Recipe type</label>
				<select id="swipe-type-filter" class="form-select" data-type-filter>
					<option value="all">All types</option>
				</select>
			</div>

			<div class="swipe-deck" data-swipe-deck aria-live="polite">
				{% for recipe in recipes %}
				<article
					class="swipe-card"
					data-swipe-card
					data-title="{{ recipe.title | escape }}"
					data-url="{{ recipe.url | relative_url }}"
					data-type="{{ recipe.recipe-type | default: 'Recipe' | escape }}"
				>
					<div class="swipe-card-image-wrap">
						{% if recipe.thumbnail %}
						<img src="{{ recipe.thumbnail }}" alt="{{ recipe.title }}" class="swipe-card-image" loading="lazy">
						{% else %}
						<div class="swipe-card-image-fallback" aria-hidden="true"></div>
						{% endif %}
					</div>
					<div class="swipe-card-badge swipe-card-badge-left" aria-hidden="true">Skip</div>
					<div class="swipe-card-badge swipe-card-badge-right" aria-hidden="true">Save</div>
					<div class="swipe-card-overlay">
						<h2 class="swipe-card-title">{{ recipe.title }}</h2>
						<div class="swipe-card-meta">
							<span class="swipe-meta-pill">{{ recipe.recipe-type | default: "Recipe" }}</span>
							{% if recipe.recipe-tags and recipe.recipe-tags.size > 0 %}
							<span class="swipe-meta-pill">{{ recipe.recipe-tags[0] }}</span>
							{% endif %}
						</div>
					</div>
				</article>
				{% endfor %}

				<div class="swipe-done" data-swipe-done>
					Deck finished. Use Undo or Reload picks to continue.
				</div>
			</div>

			<div class="swipe-controls">
				<button type="button" class="btn btn-outline-danger swipe-btn" data-action="skip">Skip</button>
				<button type="button" class="btn btn-outline-secondary swipe-btn" data-action="undo">Undo</button>
				<button type="button" class="btn btn-outline-success swipe-btn" data-action="save">Save</button>
				<button type="button" class="btn btn-primary swipe-btn" data-action="reload">Reload picks</button>
			</div>
			<p class="swipe-summary" data-swipe-summary></p>
		</div>

		<aside class="swipe-panel">
			<h2>Your shortlist</h2>
			<p class="swipe-empty" data-shortlist-empty>Saved recipes will appear here.</p>
			<ul class="swipe-shortlist" data-shortlist></ul>
			<div class="d-grid gap-2 mt-3">
				<button type="button" class="btn btn-success" data-action="random" disabled>Pick one for me</button>
			</div>
			<div class="swipe-choice d-none" data-random-choice></div>
		</aside>
	</div>
</section>

<script>
	(() => {
		const root = document.querySelector("[data-swipe-root]");
		if (!root) {
			return;
		}

		const deck = root.querySelector("[data-swipe-deck]");
		const cards = Array.from(root.querySelectorAll("[data-swipe-card]"));
		const summary = root.querySelector("[data-swipe-summary]");
		const shortlist = root.querySelector("[data-shortlist]");
		const shortlistEmpty = root.querySelector("[data-shortlist-empty]");
		const randomButton = root.querySelector('[data-action="random"]');
		const randomChoice = root.querySelector("[data-random-choice]");
		const doneBanner = root.querySelector("[data-swipe-done]");
		const typeFilter = root.querySelector("[data-type-filter]");
		const actionButtons = Array.from(root.querySelectorAll("[data-action]"));

		const likedMap = new Map();
		const history = [];
		const state = {
			dragging: false,
			locked: false,
			activeCard: null,
			startX: 0,
			startY: 0,
			deltaX: 0,
			deltaY: 0,
			selectedType: "all",
			activeCards: [],
		};

		function normalize(value) {
			return (value || "").trim().toLowerCase();
		}

		function getCardType(card) {
			return card.dataset.type || "Recipe";
		}

		function shuffleCards(list) {
			for (let i = list.length - 1; i > 0; i -= 1) {
				const j = Math.floor(Math.random() * (i + 1));
				const temp = list[i];
				list[i] = list[j];
				list[j] = temp;
			}
		}

		function getFilteredCards() {
			if (state.selectedType === "all") {
				return cards.slice();
			}
			return cards.filter((card) => normalize(getCardType(card)) === state.selectedType);
		}

		function resetCardState(card) {
			card.classList.remove("is-dismissed", "is-top", "is-skipping", "is-liking", "is-dragging");
			card.style.display = "none";
			card.style.opacity = "";
			card.style.pointerEvents = "none";
			card.style.transform = "";
			card.style.transition = "";
		}

		function rebuildDeck() {
			history.length = 0;
			likedMap.clear();
			randomChoice.classList.add("d-none");
			randomChoice.innerHTML = "";

			cards.forEach((card) => {
				resetCardState(card);
				deck.appendChild(card);
			});

			state.activeCards = getFilteredCards();
			shuffleCards(state.activeCards);

			state.activeCards.forEach((card) => {
				deck.appendChild(card);
			});

			renderShortlist();
			syncCardsUI();
		}

		function populateTypeFilter() {
			if (!typeFilter) {
				return;
			}

			const seen = new Set();
			const labels = [];
			cards.forEach((card) => {
				const label = getCardType(card).trim() || "Recipe";
				const key = normalize(label);
				if (!seen.has(key)) {
					seen.add(key);
					labels.push(label);
				}
			});
			labels.sort((a, b) => a.localeCompare(b));

			labels.forEach((label) => {
				const option = document.createElement("option");
				option.value = normalize(label);
				option.textContent = label;
				typeFilter.appendChild(option);
			});

			typeFilter.addEventListener("change", () => {
				state.selectedType = typeFilter.value || "all";
				rebuildDeck();
			});
		}

		function remainingCards() {
			return state.activeCards.filter((card) => !card.classList.contains("is-dismissed"));
		}

		function topCard() {
			const available = remainingCards();
			return available.length > 0 ? available[0] : null;
		}

		function renderShortlist() {
			shortlist.innerHTML = "";
			const likedItems = Array.from(likedMap.values());

			if (likedItems.length === 0) {
				shortlistEmpty.classList.remove("d-none");
				randomButton.disabled = true;
				randomChoice.classList.add("d-none");
				randomChoice.innerHTML = "";
				return;
			}

			shortlistEmpty.classList.add("d-none");
			randomButton.disabled = false;

			likedItems.forEach((item) => {
				const listItem = document.createElement("li");
				listItem.innerHTML = `<a href="${item.url}">${item.title}</a>`;
				shortlist.appendChild(listItem);
			});
		}

		function syncCardsUI() {
			const available = remainingCards();
			const likedCount = likedMap.size;
			const selectedLabel = typeFilter && typeFilter.selectedIndex >= 0
				? typeFilter.options[typeFilter.selectedIndex].text
				: "All types";

			if (state.activeCards.length === 0) {
				doneBanner.textContent = "No recipes found for this type.";
			} else if (available.length === 0) {
				doneBanner.textContent = "Deck finished. Use Undo or Reload picks to continue.";
			}

			doneBanner.classList.toggle("is-visible", available.length === 0);

			summary.textContent = `${available.length} recipes left in ${selectedLabel}, ${likedCount} saved.`;

			cards.forEach((card) => {
				card.classList.remove("is-top", "is-skipping", "is-liking", "is-dragging");
				card.style.opacity = "0";
				card.style.pointerEvents = "none";
				if (card.classList.contains("is-dismissed")) {
					card.style.display = "none";
					return;
				}
				card.style.display = "flex";
				card.style.transform = "translate3d(0, 0, 0)";
			});

			available.forEach((card, index) => {
				if (index > 3) {
					card.style.display = "none";
					return;
				}

				const offsetY = index * 9;
				const scale = Math.max(0.86, 1 - index * 0.035);
				card.style.opacity = String(Math.max(0.5, 1 - index * 0.14));
				card.style.transform = `translate3d(0, ${offsetY}px, 0) scale(${scale})`;
				card.style.zIndex = String(100 - index);
			});

			const currentTopCard = topCard();
			if (currentTopCard) {
				currentTopCard.classList.add("is-top");
				currentTopCard.style.pointerEvents = "auto";
			}
		}

		function clearDragStyles(card) {
			card.classList.remove("is-skipping", "is-liking", "is-dragging");
			card.style.transform = "";
			card.style.opacity = "";
		}

		function swipeCard(card, saveCard) {
			if (!card || state.locked) {
				return;
			}

			state.locked = true;
			const direction = saveCard ? 1 : -1;
			const xShift = direction * (window.innerWidth * 0.95);

			card.classList.remove("is-dragging");
			card.classList.add(saveCard ? "is-liking" : "is-skipping");
			card.style.transition = "transform 0.22s ease, opacity 0.22s ease";
			card.style.transform = `translate3d(${xShift}px, 0, 0) rotate(${direction * 24}deg)`;
			card.style.opacity = "0.1";

			window.setTimeout(() => {
				card.classList.add("is-dismissed");
				card.style.transition = "";
				clearDragStyles(card);

				const entry = {
					card,
					saved: saveCard,
					url: card.dataset.url,
					title: card.dataset.title,
				};
				history.push(entry);

				if (saveCard) {
					likedMap.set(entry.url, { title: entry.title, url: entry.url });
				}

				renderShortlist();
				syncCardsUI();
				state.locked = false;
			}, 230);
		}

		function undoLast() {
			if (state.locked) {
				return;
			}
			const last = history.pop();
			if (!last) {
				return;
			}

			if (last.saved) {
				likedMap.delete(last.url);
			}

			last.card.classList.remove("is-dismissed", "is-skipping", "is-liking", "is-dragging");
			last.card.style.display = "flex";
			last.card.style.transform = "";
			last.card.style.opacity = "";
			deck.prepend(last.card);
			renderShortlist();
			syncCardsUI();
		}

		function resetDeck() {
			if (state.locked) {
				return;
			}
			rebuildDeck();
		}

		function onPointerDown(event) {
			const active = topCard();
			if (!active || state.locked) {
				return;
			}
			if (!event.target.closest("[data-swipe-card]")) {
				return;
			}

			state.dragging = true;
			state.activeCard = active;
			state.startX = event.clientX;
			state.startY = event.clientY;
			state.deltaX = 0;
			state.deltaY = 0;

			active.classList.add("is-dragging");
			active.setPointerCapture(event.pointerId);
		}

		function onPointerMove(event) {
			if (!state.dragging || !state.activeCard) {
				return;
			}

			state.deltaX = event.clientX - state.startX;
			state.deltaY = event.clientY - state.startY;
			const angle = state.deltaX / 16;
			state.activeCard.style.transform = `translate3d(${state.deltaX}px, ${state.deltaY}px, 0) rotate(${angle}deg)`;

			state.activeCard.classList.toggle("is-liking", state.deltaX > 36);
			state.activeCard.classList.toggle("is-skipping", state.deltaX < -36);
		}

		function onPointerUp(event) {
			if (!state.dragging || !state.activeCard) {
				return;
			}

			const card = state.activeCard;
			card.releasePointerCapture(event.pointerId);
			state.dragging = false;
			state.activeCard = null;

			const threshold = Math.min(card.offsetWidth * 0.32, 130);
			if (state.deltaX > threshold) {
				swipeCard(card, true);
			} else if (state.deltaX < -threshold) {
				swipeCard(card, false);
			} else {
				card.style.transition = "transform 0.16s ease";
				card.style.transform = "translate3d(0, 0, 0) rotate(0deg)";
				window.setTimeout(() => {
					card.style.transition = "";
					clearDragStyles(card);
					syncCardsUI();
				}, 165);
			}
		}

		actionButtons.forEach((button) => {
			button.addEventListener("click", () => {
				const action = button.dataset.action;
				if (action === "skip") {
					swipeCard(topCard(), false);
					return;
				}
				if (action === "save") {
					swipeCard(topCard(), true);
					return;
				}
				if (action === "undo") {
					undoLast();
					return;
				}
				if (action === "reload") {
					resetDeck();
					return;
				}
				if (action === "random") {
					const options = Array.from(likedMap.values());
					if (options.length === 0) {
						return;
					}
					const selected = options[Math.floor(Math.random() * options.length)];
					randomChoice.classList.remove("d-none");
					randomChoice.innerHTML = `Cook this one: <a href="${selected.url}">${selected.title}</a>`;
				}
			});
		});

		document.addEventListener("keydown", (event) => {
			if (event.target && ["INPUT", "TEXTAREA", "SELECT"].includes(event.target.tagName)) {
				return;
			}
			if (event.key === "ArrowLeft") {
				event.preventDefault();
				swipeCard(topCard(), false);
			}
			if (event.key === "ArrowRight") {
				event.preventDefault();
				swipeCard(topCard(), true);
			}
			if (event.key === "z" || event.key === "Z") {
				event.preventDefault();
				undoLast();
			}
			if (event.key === "Enter") {
				const current = topCard();
				if (current && current.dataset.url) {
					window.location.href = current.dataset.url;
				}
			}
		});

		deck.addEventListener("pointerdown", onPointerDown);
		deck.addEventListener("pointermove", onPointerMove);
		deck.addEventListener("pointerup", onPointerUp);
		deck.addEventListener("pointercancel", onPointerUp);

		populateTypeFilter();
		rebuildDeck();
	})();
</script>

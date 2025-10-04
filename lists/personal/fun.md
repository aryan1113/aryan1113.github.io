---
last_modified: 2025-07-06 17:55:29 +0000
layout: default
title: Fun Screenshots
permalink: /fun/
---


Compilation of really cool and obscure shit that has happened with me online, or I was simply very lucky to observe it all
Heck I don't even remember dates for most of these, but here we go.


<h2> In no particular order </h2>

<div id="controls" style="margin-bottom: 1.5rem;">
<label for="tagFilter">Filter by tag:</label>
<select id="tagFilter">
  <option value="all">All</option>
  {% assign tags = site.data.fun | map: "tag" | uniq | sort %}
  {% for tag in tags %}
    <option value="{{ tag | downcase }}">{{ tag | capitalize }}</option>
  {% endfor %}
</select>


  <label for="sortOrder" style="margin-left: 1rem;">Sort by:</label>
  <select id="sortOrder">
    <option value="newest">Newest First</option>
    <option value="oldest">Oldest First</option>
  </select>
</div>

{% assign sorted_fun = site.data.fun | sort: "date" | reverse %}

<div class="fun-gallery" id="funGallery">
  {% for item in sorted_fun %}
  <div class="fun-item" data-tag="{{ item.tag | downcase }}" data-date="{{ item.date }}">
    <div class="image-wrapper">
        <img src="{{ item.url }}" alt="{{ item.alt }}">
    </div>
    <!-- <p class="caption">{{ item.caption }}</p> -->
    <p class="caption">{{ item.caption | newline_to_br }}</p>

    <p class="meta">
      <strong>Tag:</strong> {{ item.tag }} <br />
      <strong>Date:</strong> {{ item.date }}
    </p>
  </div>
  {% endfor %}
</div>


<script>
  const tagFilter = document.getElementById("tagFilter");
  const sortOrder = document.getElementById("sortOrder");
  const gallery = document.getElementById("funGallery");

  function updateGallery() {
    const selectedTag = tagFilter.value;
    const selectedOrder = sortOrder.value;

    // Get all items
    const allItems = Array.from(gallery.children);

    // Always reset display first
    allItems.forEach(item => {
      const tag = item.getAttribute("data-tag");
      const isMatch = (selectedTag === "all" || tag === selectedTag);
      item.style.display = isMatch ? "block" : "none";
    });

    // Now get only visible items for sorting
    const visibleItems = allItems.filter(item => item.style.display !== "none");

    // Sort visible items
    visibleItems.sort((a, b) => {
      const dateA = new Date(a.getAttribute("data-date"));
      const dateB = new Date(b.getAttribute("data-date"));
      return selectedOrder === "newest" ? dateB - dateA : dateA - dateB;
    });

    // Re-append only visible items in sorted order
    visibleItems.forEach(item => gallery.appendChild(item));
  }

  tagFilter.addEventListener("change", updateGallery);
  sortOrder.addEventListener("change", updateGallery);

  updateGallery(); // Initial render
</script>


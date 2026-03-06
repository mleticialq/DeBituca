---
layout: default
title: Categorias
permalink: /categorias/
---

<div class="categories-wrapper">

  <aside class="categories-list" id="catList">
    <!-- categorias via JS -->
  </aside>

  <main class="categories-content">
    <h2 id="catCurrentName" class="category-title">Selecione uma categoria</h2>

    <ul class="category-posts" id="catPosts">
      <!-- posts via JS -->
    </ul>
  </main>

</div>

<script>
  window.__CATEGORIES__ = (function(){
    const map = {};

    {% for category in site.categories %}
      {% assign cat_name = category[0] %}
      {% assign cat_slug = cat_name | slugify %}
      {% assign posts = category[1] %}

      map["{{ cat_slug }}"] = {
        name: {{ cat_name | jsonify }},
        count: {{ posts.size }},
        posts: [
          {% for post in posts %}
          {
            title: {{ post.title | jsonify }},
            url: {{ post.url | relative_url | jsonify }},
            dateISO: {{ post.date | date: "%Y-%m-%d" | jsonify }},
            dateBR: {{ post.date | date: "%d/%m/%Y" | jsonify }}
          }{% unless forloop.last %},{% endunless %}
          {% endfor %}
        ]
      };
    {% endfor %}

    return map;
  })();

  (function(){
    const data = window.__CATEGORIES__;
    const slugs = Object.keys(data);

    const list = document.getElementById("catList");
    const posts = document.getElementById("catPosts");
    const title = document.getElementById("catCurrentName");

    function renderCategories(active){
      list.innerHTML = "";
      slugs.sort((a,b)=>data[a].name.localeCompare(data[b].name,"pt-BR"));

      slugs.forEach(slug=>{
        const btn = document.createElement("button");
        btn.className = "category-item" + (slug === active ? " active" : "");
        btn.type = "button";
        btn.innerHTML = `
          <span class="cat-name">${data[slug].name}</span>
          <small class="cat-count">${data[slug].count}</small>
        `;
        btn.onclick = ()=>selectCategory(slug);
        list.appendChild(btn);
      });
    }

    function renderPosts(slug){
      const cat = data[slug];
      title.textContent = cat.name;

      const ordered = cat.posts.sort((a,b)=>b.dateISO.localeCompare(a.dateISO));

      posts.innerHTML = ordered.map(p=>`
        <li class="post-card">
          <a class="post-title" href="${p.url}">${p.title}</a>
          <span class="post-date">${p.dateBR}</span>
        </li>
      `).join("");

      if(!ordered.length){
        posts.innerHTML = "<li class='empty'>Sem posts nesta categoria.</li>";
      }
    }

    function selectCategory(slug){
      renderCategories(slug);
      renderPosts(slug);
    }

    if(slugs.length){
      selectCategory(slugs[0]);
    }
  })();
</script>

<style>
:root{
  --purple: #8B63D4;
  --green: #16AC92;
  --gray: #747474;
  --light-gray: #a0a0a0;
  --white: #ffffff;

  /* “clima” da foto */
  --lav-0: #fbfaff;
  --lav-1: #f4f0ff;
  --stroke: rgba(20,19,26,.08);
  --stroke-strong: rgba(20,19,26,.12);

  --shadow-soft: 0 10px 26px rgba(20,19,26,.08);
  --shadow: 0 22px 60px rgba(20,19,26,.12);
}

/* ===== Container Principal (fundo igual vibe da foto) ===== */
.categories-wrapper {
  max-width: 1100px;
  width: 95%;
  margin: 60px auto;

  border-radius: 30px;
  padding: 44px;

  /* Fundo lavanda suave + brilhos */
  background:
    radial-gradient(900px 420px at 12% 10%, rgba(139,99,212,.18), transparent 60%),
    radial-gradient(700px 420px at 92% 20%, rgba(22,172,146,.10), transparent 62%),
    linear-gradient(180deg, var(--lav-1), var(--lav-0));

  border: 1px solid var(--stroke);
  box-shadow: var(--shadow);

  display: grid;
  grid-template-columns: 300px 1fr;
  gap: 0;
  min-height: 70vh;
  align-items: start;

  position: relative;
  overflow: hidden;
}

/* brilho interno bem leve */
.categories-wrapper::before{
  content:"";
  position:absolute;
  inset:0;
  background:
    radial-gradient(600px 220px at 30% 0%, rgba(255,255,255,.75), transparent 65%),
    radial-gradient(600px 260px at 80% 100%, rgba(255,255,255,.55), transparent 70%);
  pointer-events:none;
  opacity:.55;
}

/* ===== Pilar de Categorias (esquerda) ===== */
.categories-list {
  position: relative;
  z-index: 1;

  display: flex;
  flex-direction: column;
  gap: 12px;

  padding-right: 36px;
  border-right: 2px dashed rgba(20,19,26,.10);

  /* vira “pilar” navegável */
  max-height: calc(70vh - 20px);
  overflow: auto;
  padding-top: 6px;
}

/* barra de rolagem discreta */
.categories-list::-webkit-scrollbar{ width: 10px; }
.categories-list::-webkit-scrollbar-thumb{
  background: rgba(139,99,212,.18);
  border-radius: 999px;
  border: 3px solid rgba(255,255,255,.6);
}
.categories-list::-webkit-scrollbar-track{ background: transparent; }

.category-item {
  background: rgba(255,255,255,.86);
  border: 1px solid var(--stroke);
  border-radius: 18px;
  padding: 14px 16px;

  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 14px;

  color: rgba(20,19,26,.72);
  transition: transform .18s ease, box-shadow .18s ease, border-color .18s ease, background .18s ease;

  box-shadow: 0 10px 24px rgba(20,19,26,.06);
  backdrop-filter: blur(6px);
}

.category-item .cat-name{
  font-size: 14.5px;
  letter-spacing: .2px;
  text-align: left;
}

.category-item .cat-count{
  font-size: 12px;
  opacity: .75;
  padding: 6px 10px;
  border-radius: 999px;
  border: 1px solid rgba(20,19,26,.08);
  background: rgba(255,255,255,.65);
}

.category-item:hover {
  transform: translateX(6px);
  border-color: rgba(139,99,212,.35);
  box-shadow: 0 18px 40px rgba(20,19,26,.10);
}

.category-item.active {
  background: rgba(139,99,212,.92);
  border-color: rgba(139,99,212,.50);
  color: var(--white);
}

.category-item.active .cat-count{
  background: rgba(255,255,255,.18);
  border-color: rgba(255,255,255,.28);
  color: rgba(255,255,255,.95);
}

/* ===== Pilar de Conteúdo (direita) ===== */
.categories-content {
  position: relative;
  z-index: 1;

  padding-left: 44px;
  min-width: 0;
}

.category-title {
  font-size: 40px;
  color: var(--purple);
  margin: 6px 0 34px;
  font-family: serif;
  font-weight: 600;
  letter-spacing: .2px;
}

/* ===== Lista de Posts ===== */
.category-posts {
  list-style: none;
  padding: 0;
  margin: 0;

  display: flex;
  flex-direction: column;
  gap: 22px;
}

.post-card {
  padding: 18px 18px;
  border-radius: 18px;

  background: rgba(255,255,255,.78);
  border: 1px solid rgba(20,19,26,.08);
  box-shadow: 0 14px 34px rgba(20,19,26,.08);
  backdrop-filter: blur(6px);

  display: flex;
  flex-direction: column;
  gap: 6px;
}

.post-title {
  font-size: 18px;
  font-weight: 650;
  color: rgba(20,19,26,.88);
  text-decoration: none;
  line-height: 1.25;
  transition: color .2s ease;
}

.post-title:hover {
  color: var(--purple);
}

.post-date {
  font-size: 12px;
  color: rgba(20,19,26,.45);
  text-transform: uppercase;
  letter-spacing: 1px;
}

.empty{
  padding: 14px 0;
  color: rgba(20,19,26,.55);
}

/* ===== Responsividade ===== */
@media(max-width: 900px){
  .categories-wrapper {
    grid-template-columns: 1fr;
    padding: 26px 18px;
  }

  .categories-list {
    border-right: none;
    border-bottom: 2px dashed rgba(20,19,26,.10);
    padding-right: 0;
    padding-bottom: 18px;
    margin-bottom: 18px;
    max-height: 260px;
  }

  .categories-content {
    padding-left: 0;
  }

  .category-title{
    font-size: 32px;
    margin-bottom: 22px;
  }
}
</style>

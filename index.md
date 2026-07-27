# Manimal's SPT Mods

Downloads for all my mods. The download button grabs the latest release from GitHub.

<div class="mod-grid">
{% for mod in site.data.mods %}
  <div class="mod-card">
    <img class="mod-icon" src="{{ mod.icon | relative_url }}" alt="{{ mod.name }} icon">
    <div class="mod-info">
      <h3 title="{{ mod.name }}">{{ mod.name }}</h3>
      <p>{{ mod.description }}</p>
      <a class="mod-download" href="{{ mod.repo }}/releases/latest">Download</a>
      <a class="mod-source" href="{{ mod.repo }}">Source</a>
    </div>
  </div>
{% endfor %}
</div>

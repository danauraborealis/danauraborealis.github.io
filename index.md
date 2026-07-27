<div class="mod-layout">
  <div class="mod-grid">
{% for mod in site.data.mods %}
    <div class="mod-card" {% if mod.preview %}{% if mod.preview.first %}data-preview="{% for p in mod.preview %}{% if p contains '://' %}{{ p }}{% else %}{{ p | relative_url }}{% endif %}{% unless forloop.last %}|{% endunless %}{% endfor %}"{% else %}data-preview="{{ mod.preview }}"{% endif %}{% endif %}>
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
  <div class="mod-preview" id="mod-preview">
    <span class="mod-preview-hint">Hover over a mod to see a preview</span>
  </div>
</div>

<script>
(function () {
  var panel = document.getElementById('mod-preview');
  var current = '';
  var slideTimer = null;

  function youtubeId(url) {
    var m = url.match(/(?:youtube\.com\/watch\?v=|youtu\.be\/)([\w-]{11})/);
    return m ? m[1] : null;
  }

  function render(url, loop) {
    var el;
    var yt = youtubeId(url);
    if (yt) {
      // loop needs playlist=<id> — youtube quirk for single-video loops
      el = document.createElement('iframe');
      el.src = 'https://www.youtube.com/embed/' + yt +
        '?autoplay=1&mute=1' + (loop ? '&loop=1&playlist=' + yt : '');
      el.allow = 'autoplay; encrypted-media';
      el.allowFullscreen = true;
    } else if (/\.(mp4|webm)(\?|$)/i.test(url)) {
      el = document.createElement('video');
      el.src = url;
      el.autoplay = true;
      el.muted = true;
      el.loop = loop;
      el.playsInline = true;
    } else {
      el = document.createElement('img');
      el.src = url;
      el.alt = 'mod preview';
    }
    panel.innerHTML = '';
    panel.appendChild(el);
  }

  function show(spec) {
    if (spec === current) return; // dont reload the same media on re-hover
    current = spec;
    if (slideTimer) { clearTimeout(slideTimer); slideTimer = null; }

    var urls = spec.split('|');
    if (urls.length === 1) {
      render(urls[0], true);
      return;
    }

    // sequence: images advance every 3s, a video/youtube entry plays and
    // holds — put it last and it autoplays after the slideshow
    var i = 0;
    function step() {
      var url = urls[i];
      var isMedia = youtubeId(url) || /\.(mp4|webm)(\?|$)/i.test(url);
      render(url, false);
      if (isMedia) return;
      i = (i + 1) % urls.length;
      slideTimer = setTimeout(step, 3000);
    }
    step();
  }

  document.querySelectorAll('.mod-card[data-preview]').forEach(function (card) {
    card.addEventListener('mouseenter', function () {
      show(card.dataset.preview);
    });
  });
})();
</script>

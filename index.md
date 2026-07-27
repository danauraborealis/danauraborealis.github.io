<div class="social-links">
  <a class="social-link discord" href="https://discord.gg/jcu4pgrh8X">Join the Discord</a>
  <a class="social-link kofi" href="https://ko-fi.com/manimal1920">Support me on Ko-fi</a>
</div>

<div class="mod-layout">
  <div class="mod-grid">
{% for mod in site.data.mods %}
    <div class="mod-card{% if mod.wip %} wip{% endif %}" {% if mod.preview %}{% if mod.preview.first %}data-preview="{% for p in mod.preview %}{% if p contains '://' %}{{ p }}{% else %}{{ p | relative_url }}{% endif %}{% unless forloop.last %}|{% endunless %}{% endfor %}"{% else %}data-preview="{{ mod.preview }}"{% endif %}{% endif %}>
      {% if mod.wip %}<span class="wip-tab">Under Construction!</span>{% endif %}
      {% if mod.icon %}
      <img class="mod-icon" src="{{ mod.icon | relative_url }}" alt="{{ mod.name }} icon">
      {% else %}
      <div class="mod-icon mod-icon-placeholder">?</div>
      {% endif %}
      <div class="mod-info">
        <h3 title="{{ mod.name }}">{{ mod.name }}</h3>
        <p>{{ mod.description }}</p>
        {% if mod.repo %}
        <a class="mod-download" href="{{ mod.repo }}/releases/latest">Download</a>
        <a class="mod-source" href="{{ mod.repo }}">Source</a>
        {% endif %}
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

  var SLIDE_MS = 4000;

  function isMedia(url) {
    return youtubeId(url) || /\.(mp4|webm)(\?|$)/i.test(url);
  }

  // slideshow: images slide in on a horizontal track with a progress bar
  // underneath; an optional trailing video/youtube entry plays after one
  // full pass and holds
  function showSlideshow(urls) {
    var imgs = urls.filter(function (u) { return !isMedia(u); });
    var tail = urls.filter(isMedia)[0] || null;

    var wrap = document.createElement('div');
    wrap.className = 'slideshow';
    var viewport = document.createElement('div');
    viewport.className = 'slide-viewport';
    var track = document.createElement('div');
    track.className = 'slide-track';
    var i = 0;

    // panel height follows the current image's natural aspect instead of
    // letterboxing everything into a fixed frame
    function fitHeight() {
      var img = track.children[i];
      if (img && img.naturalWidth) {
        viewport.style.height =
          (wrap.clientWidth * img.naturalHeight / img.naturalWidth) + 'px';
      }
    }

    imgs.forEach(function (src) {
      var img = document.createElement('img');
      img.src = src;
      img.alt = 'mod preview';
      img.addEventListener('load', fitHeight);
      track.appendChild(img);
    });
    var prog = document.createElement('div');
    prog.className = 'slide-progress';
    var bar = document.createElement('div');
    bar.className = 'slide-progress-bar';
    prog.appendChild(bar);
    viewport.appendChild(track);
    wrap.appendChild(viewport);
    wrap.appendChild(prog);
    panel.innerHTML = '';
    panel.appendChild(wrap);

    function step() {
      track.style.transform = 'translateX(-' + (i * 100) + '%)';
      fitHeight();
      // restart the css fill animation — clearing the inline override
      // hands control back to the class animation from frame zero
      bar.style.animation = 'none';
      void bar.offsetWidth;
      bar.style.animation = '';
      slideTimer = setTimeout(function () {
        i++;
        if (i >= imgs.length) {
          if (tail) { render(tail, false); return; }
          i = 0;
        }
        step();
      }, SLIDE_MS);
    }
    step();
  }

  function show(spec) {
    if (spec === current) return; // dont reload the same media on re-hover
    current = spec;
    if (slideTimer) { clearTimeout(slideTimer); slideTimer = null; }

    var urls = spec.split('|');
    if (urls.length === 1) {
      render(urls[0], true);
    } else {
      showSlideshow(urls);
    }
  }

  document.querySelectorAll('.mod-card[data-preview]').forEach(function (card) {
    card.addEventListener('mouseenter', function () {
      var prev = document.querySelector('.mod-card.active');
      if (prev) prev.classList.remove('active');
      card.classList.add('active');
      show(card.dataset.preview);
    });
  });
})();
</script>

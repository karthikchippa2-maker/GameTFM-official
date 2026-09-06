<script>
  import Search from "@lucide/svelte/icons/search";

  import Select from "./Select.svelte";
  import { BACKEND_OPTIONS, LOCATION_OPTIONS } from "../lib/backends.js";
  import { settings } from "../lib/settings.svelte.js";
  import { activeTab, activeUrl, open } from "../lib/tabs.svelte.js";
  import { resolve } from "../lib/url.js";

  const GOOD_MS = 100;
  const FAIR_MS = 300;

  let query = $state("");
  let latency = $state(null);

  const visible = $derived(activeTab()?.kind === "proxy" && !activeUrl());
  const label = $derived(
    latency === null ? "Checking connection…" : latency === "error" ? "Connection unavailable" : `${latency} ms`,
  );
  const level = $derived.by(() => {
    if (typeof latency !== "number") return latency === "error" ? "poor" : "";
    return latency < GOOD_MS ? "good" : latency < FAIR_MS ? "fair" : "poor";
  });

  $effect(() => {
    if (visible) measure();
  });

  async function measure() {
    latency = null;
    const started = performance.now();
    try {
      const response = await fetch("/ping", { cache: "no-store" });
      if (!response.ok) throw new Error(response.status);
      latency = Math.round(performance.now() - started);
    } catch {
      latency = "error";
    }
  }

  function keydown(event) {
    if (event.key !== "Enter") return;
    const target = resolve(query, settings.search);
    if (target) open(target);
    query = "";
  }
</script>

<section class="page" id="startPage" class:active={visible}>
  <div class="startGlow glowOne"></div>
  <div class="startGlow glowTwo"></div>

  <div class="inner">
    <div class="brandMark">
      <img src="/gametfm-logo.jpg" alt="GameTFM" />
    </div>

    <div class="brandBlock">
      <div class="brandTitle">GameTFM</div>
      <div class="brandSubtitle">A clean, fast browsing experience</div>
    </div>

    <div id="startBar" class="heroSearch">
      <Search />
      <input
        id="startSearch"
        spellcheck="false"
        autocomplete="off"
        placeholder="Search or enter a web address"
        bind:value={query}
        onkeydown={keydown}
      />
      <span class="searchHint">Enter</span>
    </div>

    <div id="startControls">
      <div class="controlCard">
        <span class="controlLabel">Backend</span>
        <Select
          id="setStartBackend"
          options={BACKEND_OPTIONS}
          value={settings.backend}
          onchange={(val) => (settings.backend = val)}
        />
      </div>
      <div class="controlCard">
        <span class="controlLabel">Location</span>
        <Select
          id="setStartLocation"
          options={LOCATION_OPTIONS}
          value={settings.location}
          onchange={(val) => (settings.location = val)}
        />
      </div>
    </div>

    <div class="startFeatures">
      <div class="featurePill"><span class="featureDot"></span>GameTFM</div>
      <div class="featurePill">Fast startup</div>
      <div class="featurePill">Minimal UI</div>
    </div>
  </div>

  <div id="latency" title="Connection status">
    <span class="dot {level}"></span>
    {label}
  </div>
</section>

<script>
  import { onMount } from "svelte";

  const api_key = "319574139c3d65012c05bc9d3e466609";

  let user = "";
  let decayTimer = null;
  let widgetVisible = true;
  let displayedSong = { title: "Now", artist: "Playing" };

  async function getTracks() {
    let track;
    try {
      const response = await fetch(
        `https://ws.audioscrobbler.com/2.0/?api_key=${api_key}&method=user.getRecentTracks&user=${user}&extended=1&limit=1&format=json`
      );
      track = (await response.json()).recenttracks.track[0];
    } catch {}
    if (!track || !track["@attr"]) {
      if (decayTimer == null) {
        decayTimer = setTimeout(() => {
          widgetVisible = false;
          decayTimer = null;
        }, 15000);
      }
    } else if (track["@attr"]) {
      displayedSong = { title: track.name, artist: track.artist.name };
      widgetVisible = true;
      if (decayTimer != null) {
        clearTimeout(decayTimer);
        decayTimer = null;
      }
    }
  }
  onMount(() => {
    const queryString = window.location.search;
    const urlParams = new URLSearchParams(queryString);
    user = urlParams.get("u");
    setInterval(getTracks, 2000);
  });
</script>

<div
  class="info flex flex-col items-start space-y-2 transition-opacity duration-500 {widgetVisible
    ? ''
    : 'opacity-0'}"
>
  <span class="row py-1 px-3">
    {displayedSong.title}
  </span>
  <span class="row py-1 px-3">
    {displayedSong.artist}
  </span>
  <span class="row py-1 px-4">
    <span class="flex">♪</span>
  </span>
</div>

<style global>
  @tailwind base;
  @tailwind components;
  @tailwind utilities;
  @import url("https://fonts.googleapis.com/css2?family=Manrope&display=swap");

  body {
    font-family: "Manrope", "Calibri";
  }

  .info .row {
    @apply bg-white font-medium text-2xl;
  }
</style>

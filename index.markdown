---
layout: default
title: SMP Streamers
---

<style>
  body {
    font-family: sans-serif;
    background: #111;
    color: #fff;
    padding: 2rem;
  }
  #streamer-container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1rem;
  }
  @media (max-width: 1024px) {
    #streamer-container {
      grid-template-columns: repeat(2, 1fr);
    }
  }
  @media (max-width: 600px) {
    #streamer-container {
      grid-template-columns: repeat(1, 1fr);
    }
  }

  .streamer {
    background: #222;
    padding: 1rem;
    border-radius: 10px;
    margin-bottom: 0;
    display: flex;
    /* Changed from space-between to start */
    justify-content: flex-start;
    align-items: center;
    cursor: pointer;
    text-decoration: none;
    color: inherit;
    min-height: 70px;
    transition: background-color 0.2s ease;
    gap: 1rem; /* add gap between image and text */
  }
  .streamer:hover {
    background: #333;
  }
  .streamer-info {
    max-width: 70%;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
  .streamer-info strong {
    font-size: 1.1rem;
    margin-bottom: 0.2rem;
  }
  .streamer-info .status {
    font-size: 0.9rem;
    color: #bbb;
  }
  .profile-pic {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    object-fit: cover;
    flex-shrink: 0;
    /* margin-right no longer needed because we use gap */
  }

  .link-box {
    display: inline-block;
    padding: 0.4rem 0.8rem;
    margin: 0.2rem;
    background: #222;
    color: #fff;
    text-decoration: none;
    border: 2px solid #555;
    border-radius: 6px;
    font-size: 0.9rem;
    font-weight: 600;
    transition: background-color 0.2s ease, border-color 0.2s ease;
    cursor: pointer;
    user-select: none;
    }

    .link-box:hover,
    .link-box:focus {
    background-color: #444;
    border-color: #aaa;
    color: #fff;
    outline: none;
    }

    .link-box:active {
    background-color: #666;
    border-color: #888;
    }

</style>

<h1>Silly SMP</h1>

<h2>Useful Links</h2>
<a href="https://map.sillysmp.net" class="link-box" target="_blank" rel="noopener noreferrer">Server Map</a>

<h2>Streamers</h2>

<div id="streamer-container">
  <p>Loading streamer status...</p>
</div>

<script>
  // List your streamer usernames here directly (or fetch from a file if you want)
  const streamerUsernames = [
    "sydderslmao",
    "anotheruser",
    "thirduser"
  ];

  async function fetchStreamData() {
    const container = document.getElementById("streamer-container");

    try {
        // Step 1: Fetch the streamer list JSON
        const resStreamers = await fetch('/assets/data/streamers.json');
        const streamers = await resStreamers.json();

        if (!Array.isArray(streamers) || streamers.length === 0) {
        container.innerHTML = "<p>No streamers found in JSON.</p>";
        return;
        }

        // Step 2: Extract usernames to build query params
        const streamerUsernames = streamers.map(s => s.username);

        // Step 3: Fetch data from your worker
        const base = "https://twitch-proxy.sydneyewens06.workers.dev";
        const params = streamerUsernames.map(name => `user_login=${name}`).join('&');
        const res = await fetch(`${base}?${params}`);
        const data = await res.json();

        if (!Array.isArray(data) || data.length === 0) {
        container.innerHTML = "<p>Error fetching Twitch data or no streamers found.</p>";
        return;
        }

        // Step 4: Sort live streamers first
        const liveStreamers = data.filter(s => s.is_live);
        const offlineStreamers = data.filter(s => !s.is_live);
        const sorted = [...liveStreamers, ...offlineStreamers];

        container.innerHTML = '';

        sorted.forEach(streamer => {
        const statusText = streamer.is_live
            ? `🟢 Live - Playing ${streamer.game_name}`
            : '🔴 Offline';

        const link = document.createElement('a');
        link.className = 'streamer';
        link.href = `https://twitch.tv/${streamer.username}`;
        link.target = '_blank';
        link.rel = 'noopener noreferrer';

        link.innerHTML = `
            <img class="profile-pic" src="${streamer.profile_image_url}" alt="${streamer.display_name} profile picture" />
            <div class="streamer-info">
            <strong>${streamer.display_name}</strong>
            <div class="status">${statusText}</div>
            </div>
        `;

        container.appendChild(link);
        });

    } catch (err) {
        container.innerHTML = `<p>Error loading streamers: ${err.message}</p>`;
        console.error(err);
    }
    }


  fetchStreamData();
</script>

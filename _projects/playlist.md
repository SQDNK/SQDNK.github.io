---
layout: page
title: playlist-generator
description: fine-tune song recommendations using Spotify's API
img:
importance: 1
category: work
---

<a href="https://github.com/SQDNK/playlist-generator">code. </a>
[React.js, Express.js, Node.js, TailwindCSS]

Motivation: Spotify's current recommendation system uses user's entire history of tracks to find
new songs. As an alternative, I wanted to see if better recommendations could be found by statistically
analyzing a group of user-chosen songs based on their audio features alone.

Results: There are associations between certain audio feature distances. For example,
using all songs in spotify's Hot Pink playlist, I find that |acousticness - energy| and
|acousticness - danceability| has a positive linear correlation with r = .73.
This is then used to calibrate the recommendation api with a range (min, target, max) for
these specific features.

Success: We can determine success by how much users liked the recommendations. This can concretely be the number of recommendations they added to their playlists and how often those songs are listened to.

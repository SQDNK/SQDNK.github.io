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

motivation: spotify's current recommendation system uses user's entire history of tracks to find
new songs. as an alternative, i wanted to see if better recommendations could be found by statistically
analyzing a group of user-chosen songs based on their audio features alone.

results: there are associations between certain audio feature distances. for example,
using all songs in spotify's Hot Pink playlist, we find that |acousticness - energy| and
|acousticness - danceability| has a positive linear correlation with r = .73.
this is used to calibrate the recommendation api with a range (min, target, max) for
these specific features.

success: we can determine success by how much users liked the recommendations,
or concretely, by how many of the recommendations they added to their playlists and how
often those songs are listened to.

(once deployed, try it here and see if it works for you :) )

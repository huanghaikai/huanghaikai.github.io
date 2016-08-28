---
layout: post
title:  "Game of Life!"
date:   2015-05-27 18:30:00
categories: recurse-center
comments_enabled: true
---

Today I paired with [Erisa][1] in order to implement the Game of Life. The code can be found [here][2].

<script src="/public/game_of_life.js"></script>

<canvas id="canvas" width="400" height="400"></canvas>

<script type="text/javascript">drawCanvas();</script>
<script type="text/javascript">setInterval(function() { 
MainLoop(); }, 1000);</script>

Initially I thought that this was a very complex problem, but if you read from [wikipedia article][3], it can be coded with only few rules. This was one of the problems which I never attacked because I thought it would be really hard.

[1]: https://github.com/erisa85
[2]: https://github.com/erisa85/GameOfLife
[3]: http://en.wikipedia.org/wiki/Conway%27s_Game_of_Life
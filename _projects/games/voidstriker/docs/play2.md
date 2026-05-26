---
layout: base
title: VOID STRIKER
permalink: /voidstriker/tracker
description: A parallax space shooter built on GameEnginev1.1 — mouse-aimed continuous fire variant.
---

<div id="voidstriker-container" style="position:relative; width:100%; max-width:1025px; height:570px; margin:0 auto; background:#000; border:1px solid #00eeff; box-shadow:0 0 20px rgba(0,200,255,0.3);"></div>

<script type="module">
  import Game        from '{{site.baseurl}}/assets/js/GameEnginev1.1/essentials/Game.js';
  import GameControl from '{{site.baseurl}}/assets/js/GameEnginev1.1/essentials/GameControl.js';
  import GameLevelVoidStrikerMouse from '{{site.baseurl}}/assets/js/projects/voidstriker/levels/GameLevelVoidStrikerMouse.js';

  const gameContainer = document.getElementById('voidstriker-container');

  Game.main({
    path: '{{site.baseurl}}',
    gameContainer,
    gameLevelClasses: [GameLevelVoidStrikerMouse],
    innerWidth:  gameContainer.clientWidth  || 1025,
    innerHeight: gameContainer.clientHeight || 570,
    disablePauseMenu: true,
    disableContainerAdjustment: true
  }, GameControl);
</script>

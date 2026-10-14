<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="color-scheme" content="dark">
  <title>Simulador de dado</title>

  <style>
    * { box-sizing: border-box; }

    body {
      margin: 0;
      min-height: 100vh;
      background: #100b08;
      color: #f7edda;
      font-family: system-ui, -apple-system, "Segoe UI", sans-serif;
    }

    main {
      max-width: 1240px;
      margin: auto;
      padding: 48px 28px 64px;
    }

    .eyebrow {
      color: #d4ac70;
      font-size: 12px;
      letter-spacing: .18em;
      text-transform: uppercase;
    }

    h1 {
      font-family: Georgia, serif;
      font-size: clamp(36px, 5vw, 56px);
      letter-spacing: -.04em;
      margin: 12px 0;
      font-weight: 400;
    }

    .intro {
      color: #c0ad94;
      line-height: 1.6;
      margin: 0;
    }

    .layout {
      display: grid;
      grid-template-columns: 1.4fr 1fr;
      gap: 32px;
      margin-top: 32px;
      align-items: stretch;
    }

    .play,
    .stats {
      border: 1px solid #c49a5933;
      border-radius: 24px;
    }

    .play {
      position: relative;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-end;
      min-height: 620px;
      padding: 0 0 26px;
      background: linear-gradient(#130d0870, #1d130bc0);
      overflow: hidden;
    }

    .stats {
      padding: 32px;
      background: #17120dd9;
      backdrop-filter: blur(18px);
      -webkit-backdrop-filter: blur(18px);
      box-shadow: 0 24px 70px #0004, inset 0 1px #f7dfaa0c;
      align-self: center;
    }

    #scene {
      width: 100%;
      height: 420px;
      position: relative;
      border-radius: 24px 24px 0 0;
      overflow: hidden;
    }

    #scene canvas {
      display: block;
      width: 100%;
      height: 100%;
    }

    .die {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      grid-template-rows: repeat(3, 1fr);
      gap: 12%;
      width: 190px;
      aspect-ratio: 1;
      padding: 36px;
      border-radius: 22%;
      background: #f0f0f2;
      box-shadow: 0 14px 30px #0004, inset 0 -6px 0 #d2d3d8;
    }

    #dado {
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
    }

    #scene.ready #dado {
      width: 1px;
      height: 1px;
      padding: 0;
      overflow: hidden;
      clip-path: inset(50%);
    }

    .pip {
      border-radius: 50%;
      background: #25262b;
      visibility: hidden;
    }

    .pip.visible { visibility: visible; }

    .die.small {
      width: 32px;
      flex-shrink: 0;
      gap: 10%;
      padding: 6px;
      box-shadow: none;
      border-radius: 7px;
    }

    .scene-note {
      font-size: 12px;
      color: #d9c093;
      padding: 0 22px;
      text-align: center;
    }

    .scene-note:empty { display: none; }

    #resultado {
      color: #eadbc3;
      min-height: 24px;
      margin: 8px 0 20px;
      padding: 0 20px;
      text-align: center;
    }

    button {
      font: inherit;
      cursor: pointer;
      border-radius: 12px;
      transition: background .2s, transform .2s;
    }

    #jogar {
      background: linear-gradient(135deg, #e8c78c, #b98a48);
      color: #261a0e;
      border: 1px solid #f6deb066;
      box-shadow: 0 4px 24px #0005, inset 0 1px #fff5;
      padding: 15px 32px;
      font-weight: 650;
      width: min(100%, 250px);
    }

    #jogar:hover:not(:disabled) {
      background: linear-gradient(135deg, #f3d8a8, #ce9e5d);
      transform: translateY(-2px);
    }

    button:focus-visible {
      outline: 3px solid #efcd90;
      outline-offset: 5px;
    }

    button:disabled {
      cursor: wait;
      opacity: .55;
    }

    .hint {
      color: #ba9f7b;
      font-size: 12px;
      line-height: 1.7;
      margin-top: 18px;
      padding: 0 18px;
      text-align: center;
    }

    h2 {
      font-size: 19px;
      letter-spacing: -.02em;
      margin: 0;
    }

    .total {
      display: flex;
      align-items: baseline;
      justify-content: space-between;
      margin: 24px 0;
      padding-bottom: 24px;
      border-bottom: 1px solid #c49a592b;
      color: #bdac94;
      font-size: 14px;
    }

    #total {
      color: #f4dfb6;
      font-size: 32px;
      font-weight: 600;
      font-variant-numeric: tabular-nums;
    }

    .rows {
      display: grid;
      gap: 20px;
    }

    .row {
      display: grid;
      grid-template-columns: 32px 1fr;
      gap: 14px;
      align-items: center;
    }

    .row-info {
      display: flex;
      justify-content: space-between;
      gap: 8px;
      font-size: 13px;
      margin-bottom: 8px;
      color: #e1d1b6;
      font-variant-numeric: tabular-nums;
    }

    .percent { color: #baa78d; }

    .track {
      height: 5px;
      background: #a17b4326;
      border-radius: 6px;
      overflow: hidden;
    }

    .bar {
      height: 100%;
      width: 0;
      border-radius: inherit;
      background: linear-gradient(90deg, #a57535, #edcb8a);
      box-shadow: 0 0 12px #dca44a30;
      transition: width .4s ease;
    }

    #reiniciar {
      display: block;
      margin: 28px auto 0;
      padding: 9px 12px;
      color: #c4ae8d;
      background: transparent;
      border: 1px solid #b38d5440;
      font-size: 12px;
    }

    #reiniciar:hover:not(:disabled) {
      color: white;
      background: #b38d5415;
    }

    #aviso {
      color: #d8bd86;
      font-size: 13px;
      line-height: 1.5;
      margin-top: 20px;
    }

    #aviso:empty { display: none; }

    .bar-backdrop {
      position: fixed;
      inset: 0;
      z-index: -1;
      overflow: hidden;
      background:
        radial-gradient(ellipse at 20% 20%, #71452380, transparent 45%),
        linear-gradient(#130e0b 0%, #24190f 48%, #110b08 100%);
    }

    .shelves {
      position: absolute;
      inset: 0 0 25%;
      opacity: .65;
      filter: blur(7px);
      background: repeating-linear-gradient(
        0deg,
        transparent 0 115px,
        #090706 116px 128px,
        #a5652c55 130px 133px,
        transparent 135px 260px
      );
    }

    .bottle {
      position: absolute;
      bottom: 33%;
      width: 55px;
      height: 170px;
      border-radius: 22px 22px 9px 9px;
      background: linear-gradient(
        90deg, #18130c, #9c632750 25%, #3f3019 60%, #120f09
      );
      border: 1px solid #b38b4b44;
      box-shadow: inset 5px 0 8px #eec28522;
    }

    .bottle::before {
      content: '';
      position: absolute;
      width: 20px;
      height: 50px;
      top: -40px;
      left: 16px;
      border-radius: 4px;
      background: linear-gradient(90deg, #21180e, #73522c, #17100a);
    }

    .bottle::after {
      content: '';
      position: absolute;
      inset: 70px 5px 35px;
      background: #b19a6c77;
      border-radius: 2px;
    }

    .bottle:nth-child(1) { left: 8%; transform: scale(.85); }
    .bottle:nth-child(2) { left: 20%; }
    .bottle:nth-child(3) { left: 72%; transform: scale(1.2); }
    .bottle:nth-child(4) { left: 87%; transform: scale(.8); }

    .ambient {
      position: absolute;
      width: 140px;
      height: 140px;
      top: 4%;
      border-radius: 50%;
      background: #ffb34a;
      filter: blur(65px);
      opacity: .18;
    }

    .ambient:last-child { right: 10%; }

    .rolling { animation: roll .7s ease-in-out; }

    @keyframes roll {
      0% { transform: translate(-50%, -50%) rotate(0); }
      25% { transform: translate(-50%, -60%) rotate(-18deg); }
      60% { transform: translate(-50%, -50%) rotate(20deg); }
      100% { transform: translate(-50%, -50%) rotate(0); }
    }

    @media (max-width: 760px) {
      main { padding: 30px 16px; }
      .layout { grid-template-columns: 1fr; gap: 20px; }
      .play { min-height: 500px; }
      #scene { height: 340px; }
      .stats { width: 100%; padding: 26px; }
    }

    @media (prefers-reduced-motion: reduce) {
      *, *::before, *::after {
        animation: none !important;
        transition: none !important;
      }
    }
  </style>
</head>

<body>
  <div class="bar-backdrop" aria-hidden="true">
    <div class="shelves">
      <i class="bottle"></i>
      <i class="bottle"></i>
      <i class="bottle"></i>
      <i class="bottle"></i>
    </div>
    <div class="ambient"></div>
    <div class="ambient"></div>
  </div>

  <main>
    <header>
      <span class="eyebrow">A mesa do acaso · edição âmbar</span>
      <h1>Simulador de dado</h1>
      <p class="intro">
        Um dado de marfim, uma mesa de madeira e um novo lance.
      </p>
    </header>

    <div class="layout">
      <section class="play" aria-label="Lançar o dado">
        <div id="scene">
          <div
            id="dado"
            class="die"
            role="img"
            aria-label="Dado mostrando a face 1"
          ></div>
        </div>

        <p id="scene-note" class="scene-note" role="status"></p>

        <p
          id="resultado"
          role="status"
          aria-live="polite"
          aria-atomic="true"
        >
          Tudo pronto. Faça seu primeiro lançamento.
        </p>

        <button id="jogar" type="button">Jogar dado</button>

        <span class="hint">
          Passe o cursor sobre o dado ou clique e arraste para movê-lo.
          <br>
          Cada face tem a mesma chance de sair.
        </span>
      </section>

      <section class="stats" aria-labelledby="titulo-estatisticas">
        <h2 id="titulo-estatisticas">Estatísticas dos lançamentos</h2>

        <div class="total">
          <span>Total de lançamentos</span>
          <strong id="total">0</strong>
        </div>

        <div id="faces" class="rows"></div>

        <button id="reiniciar" type="button">
          Reiniciar estatísticas
        </button>

        <p id="aviso" role="status"></p>
      </section>
    </div>
  </main>

  <script
    src="https://cdn.jsdelivr.net/npm/three@0.146.0/build/three.min.js"
    crossorigin="anonymous"
  ></script>

  <script>
    const STORAGE_KEY = 'simulador-dado-estatisticas-v1';

    const positions = [
      [],
      [4],
      [0, 8],
      [0, 4, 8],
      [0, 2, 6, 8],
      [0, 2, 4, 6, 8],
      [0, 2, 3, 5, 6, 8]
    ];

    const die = document.getElementById('dado');
    const rollButton = document.getElementById('jogar');
    const resetButton = document.getElementById('reiniciar');
    const result = document.getElementById('resultado');
    const warning = document.getElementById('aviso');

    let counts = [0, 0, 0, 0, 0, 0];
    let lastFace = null;
    let rolling = false;
    let diceView = null;

    function drawFace(element, face) {
      if (element === die && diceView) {
        diceView.show(face);
        element.setAttribute(
          'aria-label',
          'Face superior do dado: ' + face
        );
        return;
      }

      element.replaceChildren();

      for (let index = 0; index < 9; index++) {
        const pip = document.createElement('span');
        pip.className =
          'pip' + (positions[face].includes(index) ? ' visible' : '');
        element.appendChild(pip);
      }

      element.setAttribute(
        'aria-label',
        'Dado mostrando a face ' + face
      );
    }

    const rows = counts.map((_, index) => {
      const face = index + 1;
      const row = document.createElement('div');
      row.className = 'row';

      row.innerHTML = `
        <div class="die small" role="img"></div>
        <div>
          <div class="row-info">
            <span>
              Face ${face} · <span class="count">0</span> vezes
            </span>
            <span class="percent">0,0%</span>
          </div>
          <div class="track">
            <div class="bar"></div>
          </div>
        </div>
      `;

      drawFace(row.querySelector('.die'), face);
      document.getElementById('faces').appendChild(row);
      return row;
    });

    function updateStats() {
      const total = counts.reduce((sum, count) => sum + count, 0);

      document.getElementById('total').textContent =
        total.toLocaleString('pt-BR');

      rows.forEach((row, index) => {
        const percentage =
          total === 0 ? 0 : counts[index] / total * 100;

        row.querySelector('.count').textContent =
          counts[index].toLocaleString('pt-BR');

        row.querySelector('.percent').textContent =
          percentage.toLocaleString('pt-BR', {
            minimumFractionDigits: 1,
            maximumFractionDigits: 1
          }) + '%';

        row.querySelector('.bar').style.width = percentage + '%';
      });
    }

    function saveStats() {
      try {
        localStorage.setItem(
          STORAGE_KEY,
          JSON.stringify({ counts, lastFace })
        );
        warning.textContent = '';
      } catch {
        warning.textContent =
          'O navegador não permitiu salvar os dados. ' +
          'As estatísticas continuam funcionando nesta sessão.';
      }
    }

    try {
      const stored = localStorage.getItem(STORAGE_KEY);

      if (stored !== null) {
        const data = JSON.parse(stored);

        if (
          !data ||
          !Array.isArray(data.counts) ||
          data.counts.length !== 6 ||
          !data.counts.every(count =>
            Number.isSafeInteger(count) && count >= 0
          ) ||
          !Number.isSafeInteger(
            data.counts.reduce((sum, count) => sum + count, 0)
          ) ||
          !(
            data.lastFace === null ||
            (
              Number.isInteger(data.lastFace) &&
              data.lastFace >= 1 &&
              data.lastFace <= 6
            )
          )
        ) {
          throw new Error('Dados inválidos');
        }

        counts = data.counts;
        lastFace = data.lastFace;
      }
    } catch {
      warning.textContent =
        'Não foi possível recuperar as estatísticas salvas. ' +
        'Uma nova contagem foi iniciada.';
    }

    drawFace(die, lastFace || 1);

    if (lastFace !== null) {
      result.textContent = 'Último resultado: face ' + lastFace + '.';
    }

    updateStats();

    function createDiceView() {
      if (!window.THREE) {
        throw new Error('Three.js indisponível');
      }

      const T = window.THREE;
      const host = document.getElementById('scene');

      const renderer = new T.WebGLRenderer({
        antialias: true,
        alpha: false
      });

      renderer.setPixelRatio(
        Math.min(window.devicePixelRatio || 1, 1.75)
      );

      renderer.outputEncoding = T.sRGBEncoding;
      renderer.toneMapping = T.ACESFilmicToneMapping;
      renderer.toneMappingExposure = .85;
      renderer.shadowMap.enabled = true;
      renderer.shadowMap.type = T.PCFSoftShadowMap;

      host.appendChild(renderer.domElement);
      renderer.domElement.setAttribute('aria-hidden', 'true');

      const scene = new T.Scene();
      scene.background = new T.Color('#21160e');
      scene.fog = new T.FogExp2('#21160e', .055);

      const camera = new T.PerspectiveCamera(38, 1, .1, 60);
      camera.position.set(5.7, 5.7, 8.5);
      camera.lookAt(0, .8, 0);

      // Iluminação refletida no dado e nos copos.
      const environment = new T.Scene();
      environment.background = new T.Color('#21170f');

      const panels = [
        [-4, 5, 2, 0xffe4bb, 2.5],
        [4, 3, -2, 0xffb867, 1.3],
        [0, 6, -5, 0xffffff, 1.4]
      ];

      for (const [x, y, z, color, power] of panels) {
        const panel = new T.Mesh(
          new T.PlaneGeometry(5, 3),
          new T.MeshBasicMaterial({ color, toneMapped: false })
        );

        panel.position.set(x, y, z);
        panel.lookAt(0, 0, 0);
        panel.material.color.multiplyScalar(power);
        environment.add(panel);
      }

      const pmrem = new T.PMREMGenerator(renderer);
      const environmentMap = pmrem.fromScene(environment, .08);

      scene.environment = environmentMap.texture;
      pmrem.dispose();

      scene.add(
        new T.HemisphereLight(0xffe3b9, 0x21120b, .65)
      );

      const key = new T.DirectionalLight(0xffe4bd, 1.6);
      key.position.set(-3, 8, 5);
      key.castShadow = true;
      key.shadow.mapSize.set(1024, 1024);
      key.shadow.camera.left = -7;
      key.shadow.camera.right = 7;
      key.shadow.camera.top = 7;
      key.shadow.camera.bottom = -7;
      key.shadow.camera.near = .5;
      key.shadow.camera.far = 25;
      key.shadow.normalBias = .025;
      key.shadow.bias = -.0003;
      key.shadow.radius = 3;
      scene.add(key);

      const rim = new T.PointLight(0xffa544, 22, 25, 2);
      rim.position.set(2, 5, -5);
      scene.add(rim);

      function woodTexture() {
        const canvas = document.createElement('canvas');
        canvas.width = 1024;
        canvas.height = 1024;

        const ctx = canvas.getContext('2d');
        ctx.fillStyle = '#49301d';
        ctx.fillRect(0, 0, 1024, 1024);

        let seed = 81;

        const rnd = () => {
          seed = (seed * 1664525 + 1013904223) >>> 0;
          return seed / 4294967296;
        };

        for (let i = 0; i < 2600; i++) {
          const y = rnd() * 1024;
          const wave = rnd() * 14 + 2;
          const phase = rnd() * 6.28;

          ctx.strokeStyle =
            `rgba(${rnd() > .5 ? '157,104,52' : '14,8,4'},${rnd() * .21})`;

          ctx.lineWidth = rnd() * 2 + .3;
          ctx.beginPath();

          for (let x = 0; x <= 1024; x += 16) {
            const yy =
              y +
              Math.sin(x * .008 + phase) * wave +
              Math.sin(x * .023 + phase) * 2;

            if (x === 0) ctx.moveTo(x, yy);
            else ctx.lineTo(x, yy);
          }

          ctx.stroke();
        }

        for (let y = 0; y < 1024; y += 256) {
          ctx.fillStyle = '#0d080780';
          ctx.fillRect(0, y, 1024, 3);
        }

        for (let i = 0; i < 70; i++) {
          ctx.strokeStyle = '#dac09918';
          ctx.beginPath();

          const x = rnd() * 1024;
          const y = rnd() * 1024;

          ctx.moveTo(x, y);
          ctx.lineTo(x + 20 + rnd() * 90, y + 2);
          ctx.stroke();
        }

        const texture = new T.CanvasTexture(canvas);
        texture.wrapS = texture.wrapT = T.RepeatWrapping;
        texture.repeat.set(2, 2);
        texture.anisotropy = Math.min(
          8,
          renderer.capabilities.getMaxAnisotropy()
        );

        return texture;
      }

      const wood = woodTexture();
      wood.encoding = T.sRGBEncoding;

      const table = new T.Mesh(
        new T.BoxGeometry(30, .2, 30),
        new T.MeshStandardMaterial({
          color: 0x9a795d,
          map: wood,
          bumpMap: wood,
          bumpScale: .055,
          roughness: .52,
          metalness: .015
        })
      );

      table.position.y = -.1;
      table.receiveShadow = true;
      scene.add(table);

      const ivoryMaps = [];

      for (let face = 1; face <= 6; face++) {
        const canvas = document.createElement('canvas');
        canvas.width = canvas.height = 512;

        const ctx = canvas.getContext('2d');
        const fill = ctx.createLinearGradient(0, 0, 512, 512);

        fill.addColorStop(0, '#fff8e6');
        fill.addColorStop(1, '#dcd0b5');
        ctx.fillStyle = fill;
        ctx.fillRect(0, 0, 512, 512);

        const depth = document.createElement('canvas');
        depth.width = depth.height = 512;

        const bump = depth.getContext('2d');
        bump.fillStyle = '#cccccc';
        bump.fillRect(0, 0, 512, 512);

        for (const pos of positions[face]) {
          const x = 256 + ((pos % 3) - 1) * 116;
          const y = 256 + (Math.floor(pos / 3) - 1) * 116;

          const pit = ctx.createRadialGradient(
            x - 5, y - 6, 2, x, y, 35
          );

          pit.addColorStop(0, '#34302a');
          pit.addColorStop(.65, '#12100e');
          pit.addColorStop(.88, '#090807');
          pit.addColorStop(1, '#7e725c');

          ctx.fillStyle = pit;
          ctx.beginPath();
          ctx.arc(x, y, 35, 0, Math.PI * 2);
          ctx.fill();

          const depression = bump.createRadialGradient(
            x, y, 0, x, y, 36
          );

          depression.addColorStop(0, '#111111');
          depression.addColorStop(.7, '#333333');
          depression.addColorStop(1, '#cccccc');

          bump.fillStyle = depression;
          bump.beginPath();
          bump.arc(x, y, 36, 0, Math.PI * 2);
          bump.fill();
        }

        const map = new T.CanvasTexture(canvas);
        map.encoding = T.sRGBEncoding;

        const bumpMap = new T.CanvasTexture(depth);

        ivoryMaps[face] = new T.MeshPhysicalMaterial({
          map,
          bumpMap,
          bumpScale: .075,
          roughness: .22,
          metalness: 0,
          clearcoat: .55,
          clearcoatRoughness: .22,
          envMapIntensity: .65
        });
      }

      // As faces opostas somam sete.
      const faceNumbers = [2, 5, 1, 6, 3, 4];

      const normals = [
        new T.Vector3(0, 1, 0),
        new T.Vector3(1, 0, 0),
        new T.Vector3(0, 0, 1),
        new T.Vector3(0, 0, -1),
        new T.Vector3(-1, 0, 0),
        new T.Vector3(0, -1, 0)
      ];

      const geometry = new T.BoxGeometry(2, 2, 2, 24, 24, 24);
      const vertex = geometry.attributes.position;
      const v = new T.Vector3();
      const inner = new T.Vector3();

      for (let i = 0; i < vertex.count; i++) {
        v.fromBufferAttribute(vertex, i);
        inner.copy(v).clampScalar(-.86, .86);

        v.sub(inner)
          .normalize()
          .multiplyScalar(.14)
          .add(inner);

        vertex.setXYZ(i, v.x, v.y, v.z);
      }

      geometry.computeVertexNormals();

      const cube = new T.Mesh(
        geometry,
        faceNumbers.map(face => ivoryMaps[face])
      );

      cube.castShadow = true;
      cube.receiveShadow = true;
      cube.position.y = 1;
      scene.add(cube);

      // Aberturas e cavidades reais para os pontos.
      const indices = geometry.index.array;
      const kept = [];
      const groups = geometry.groups.map(group => ({ ...group }));

      geometry.clearGroups();

      for (const group of groups) {
        const start = kept.length;

        const axisIndex =
          group.materialIndex < 2 ? 0 :
          group.materialIndex < 4 ? 1 : 2;

        const coordinate = index =>
          axisIndex === 0 ? vertex.getX(index) :
          axisIndex === 1 ? vertex.getY(index) :
          vertex.getZ(index);

        for (
          let i = group.start;
          i < group.start + group.count;
          i += 3
        ) {
          const flat = [
            indices[i],
            indices[i + 1],
            indices[i + 2]
          ].every(index =>
            Math.abs(Math.abs(coordinate(index)) - 1) < .00001
          );

          if (!flat) {
            kept.push(indices[i], indices[i + 1], indices[i + 2]);
          }
        }

        geometry.addGroup(
          start,
          kept.length - start,
          group.materialIndex
        );
      }

      geometry.setIndex(kept);

      const faceMaterial = new T.MeshPhysicalMaterial({
        color: 0xf5ead1,
        roughness: .29,
        clearcoat: .6,
        clearcoatRoughness: .2,
        envMapIntensity: .55
      });

      const pitMaterial = new T.MeshStandardMaterial({
        color: 0x090807,
        roughness: .48,
        metalness: 0
      });

      for (let face = 1; face <= 6; face++) {
        const shape = new T.Shape();
        const edge = 5 / 6;

        shape.moveTo(-edge, -edge);
        shape.lineTo(edge, -edge);
        shape.lineTo(edge, edge);
        shape.lineTo(-edge, edge);
        shape.closePath();

        const surface = new T.Group();

        surface.quaternion.setFromUnitVectors(
          new T.Vector3(0, 0, 1),
          normals[face - 1]
        );

        for (const pos of positions[face]) {
          const x = ((pos % 3) - 1) * .453125;
          const y = (1 - Math.floor(pos / 3)) * .453125;
          const radius = .137;

          const hole = new T.Path();
          hole.absarc(x, y, radius, 0, Math.PI * 2, true);
          shape.holes.push(hole);

          const points = [x, y, .922];
          const triangles = [];
          const rings = 5;
          const segments = 40;

          for (let ring = 1; ring <= rings; ring++) {
            const r = radius * ring / rings;

            for (let segment = 0; segment < segments; segment++) {
              const angle = segment / segments * Math.PI * 2;

              points.push(
                x + Math.cos(angle) * r,
                y + Math.sin(angle) * r,
                1 - .078 * (1 - (r / radius) * (r / radius))
              );
            }
          }

          for (let segment = 0; segment < segments; segment++) {
            triangles.push(
              0,
              1 + segment,
              1 + (segment + 1) % segments
            );
          }

          for (let ring = 0; ring < rings - 1; ring++) {
            for (let segment = 0; segment < segments; segment++) {
              const a = 1 + ring * segments + segment;
              const b = 1 + ring * segments +
                (segment + 1) % segments;
              const c = a + segments;
              const d = b + segments;

              triangles.push(a, c, b, b, c, d);
            }
          }

          const bowlGeometry = new T.BufferGeometry();

          bowlGeometry.setAttribute(
            'position',
            new T.Float32BufferAttribute(points, 3)
          );

          bowlGeometry.setIndex(triangles);
          bowlGeometry.computeVertexNormals();

          const bowl = new T.Mesh(bowlGeometry, pitMaterial);
          surface.add(bowl);
        }

        const faceMesh = new T.Mesh(
          new T.ShapeGeometry(shape, 40),
          faceMaterial
        );

        faceMesh.position.z = 1;
        faceMesh.receiveShadow = true;
        faceMesh.castShadow = true;

        surface.add(faceMesh);
        cube.add(surface);
      }

      // Copos e bebidas decorativos.
      const glass = new T.MeshPhysicalMaterial({
        color: 0xffeed2,
        transparent: true,
        opacity: .24,
        roughness: .08,
        metalness: .08,
        side: T.DoubleSide,
        depthWrite: false,
        clearcoat: 1,
        envMapIntensity: 1.8
      });

      const whiskey = new T.MeshPhysicalMaterial({
        color: 0x9e480c,
        transparent: true,
        opacity: .85,
        roughness: .16,
        metalness: 0,
        clearcoat: 1
      });

      function tumbler(x, z) {
        const group = new T.Group();
        group.position.set(x, 0, z);

        const points = [
          new T.Vector2(.5, .07),
          new T.Vector2(.56, .12),
          new T.Vector2(.61, 1.35),
          new T.Vector2(.55, 1.35),
          new T.Vector2(.5, .18),
          new T.Vector2(0, .18)
        ];

        const cup = new T.Mesh(
          new T.LatheGeometry(points, 48),
          glass
        );

        group.add(cup);

        const drink = new T.Mesh(
          new T.CylinderGeometry(.53, .49, .54, 48),
          whiskey
        );

        drink.position.y = .48;
        group.add(drink);

        const edge = new T.Mesh(
          new T.TorusGeometry(.58, .018, 8, 48),
          new T.MeshStandardMaterial({
            color: 0xc5ad89,
            metalness: .7,
            roughness: .13
          })
        );

        edge.rotation.x = Math.PI / 2;
        edge.position.y = 1.35;
        group.add(edge);

        for (let i = 0; i < 2; i++) {
          const ice = new T.Mesh(
            new T.BoxGeometry(.35, .32, .35),
            new T.MeshPhysicalMaterial({
              color: 0xfff4d9,
              transparent: true,
              opacity: .28,
              roughness: .2,
              clearcoat: 1
            })
          );

          ice.position.set(i * .25 - .12, .75, i * .2 - .1);
          ice.rotation.y = i + .4;
          group.add(ice);
        }

        scene.add(group);
      }

      tumbler(-3.2, -2.3);
      tumbler(3.4, -3.7);

      const shelf = new T.Mesh(
        new T.BoxGeometry(20, .18, 1.8),
        new T.MeshStandardMaterial({
          color: 0x211309,
          roughness: .5
        })
      );

      shelf.position.set(0, 1.3, -8);
      scene.add(shelf);

      for (let i = 0; i < 9; i++) {
        const bottle = new T.Group();
        bottle.position.set((i - 4) * 1.5, 1.4, -8);

        const body = new T.Mesh(
          new T.LatheGeometry([
            new T.Vector2(0, 0),
            new T.Vector2(.34, 0),
            new T.Vector2(.36, .1),
            new T.Vector2(.36, 1.3),
            new T.Vector2(.13, 1.6),
            new T.Vector2(.13, 2.15),
            new T.Vector2(0, 2.15)
          ], 32),
          new T.MeshPhysicalMaterial({
            color: i % 2 ? 0x324128 : 0x785021,
            roughness: .18,
            metalness: .08,
            clearcoat: 1
          })
        );

        bottle.add(body);

        const label = new T.Mesh(
          new T.CylinderGeometry(.365, .365, .5, 32),
          new T.MeshStandardMaterial({
            color: i % 3 ? 0xbca375 : 0x241910,
            roughness: .7
          })
        );

        label.position.y = .8;
        bottle.add(label);
        scene.add(bottle);
      }

      for (const x of [-5, 4]) {
        const lamp = new T.Mesh(
          new T.SphereGeometry(.15, 16, 12),
          new T.MeshBasicMaterial({ color: 0xffcb78 })
        );

        lamp.position.set(x, 4, -7);
        scene.add(lamp);

        const light = new T.PointLight(0xffb75d, 12, 10, 2);
        light.position.copy(lamp.position);
        scene.add(light);
      }

      let animation = null;
      let frame = null;
      let yaw = .32;

      const up = new T.Vector3(0, 1, 0);

      function orientation(face, angle) {
        const q = new T.Quaternion().setFromUnitVectors(
          normals[face - 1],
          up
        );

        return new T.Quaternion()
          .setFromAxisAngle(up, angle)
          .multiply(q);
      }

      // Profundidade de campo.
      const target = new T.WebGLRenderTarget(1, 1, {
        depthBuffer: true
      });

      target.depthTexture = new T.DepthTexture(1, 1);

      const postScene = new T.Scene();
      const postCamera = new T.OrthographicCamera(
        -1, 1, 1, -1, 0, 1
      );

      const postMaterial = new T.ShaderMaterial({
        uniforms: {
          image: { value: target.texture },
          depth: { value: target.depthTexture },
          resolution: { value: new T.Vector2(1, 1) },
          near: { value: camera.near },
          far: { value: camera.far },
          focus: {
            value: camera.position.distanceTo(
              new T.Vector3(0, 1, 0)
            )
          },
          exposure: { value: .38 }
        },

        vertexShader: `
          varying vec2 uvCoord;
          void main() {
            uvCoord = uv;
            gl_Position = vec4(position.xy, 0.0, 1.0);
          }
        `,

        fragmentShader: `
          uniform sampler2D image;
          uniform sampler2D depth;
          uniform vec2 resolution;
          uniform float near;
          uniform float far;
          uniform float focus;
          uniform float exposure;
          varying vec2 uvCoord;

          float distanceAt(vec2 uv) {
            float d = texture2D(depth, uv).x;
            return near * far / (far - d * (far - near));
          }

          void main() {
            float center = distanceAt(uvCoord);
            float blur = clamp(
              (center - focus - 1.0) * 1.4, 0.0, 5.0
            );

            vec3 color = texture2D(image, uvCoord).rgb;
            float weight = 1.0;

            for (int i = 0; i < 16; i++) {
              float a = float(i) * 2.399963;
              float r = sqrt((float(i) + .5) / 16.0);

              vec2 offset =
                vec2(cos(a), sin(a)) * r * blur / resolution;

              vec2 sampleUV = clamp(
                uvCoord + offset, vec2(.001), vec2(.999)
              );

              float accepted = step(
                center - 2.0, distanceAt(sampleUV)
              );

              color +=
                texture2D(image, sampleUV).rgb * accepted;
              weight += accepted;
            }

            color = color / weight * exposure;

            color = clamp(
              (color * (2.51 * color + .03)) /
              (color * (2.43 * color + .59) + .14),
              0.0, 1.0
            );

            color = pow(color, vec3(1.0 / 2.2));

            float vignette = 1.0 -
              .22 * pow(length((uvCoord - .5) * 1.3), 2.0);

            gl_FragColor = vec4(color * vignette, 1.0);
          }
        `,

        depthTest: false,
        depthWrite: false,
        toneMapped: false
      });

      postScene.add(
        new T.Mesh(new T.PlaneGeometry(2, 2), postMaterial)
      );

      function render() {
        renderer.setRenderTarget(target);
        renderer.render(scene, camera);
        renderer.setRenderTarget(null);
        renderer.render(postScene, postCamera);
      }

      function supportHeight() {
        const y = new T.Vector3(0, 1, 0).applyQuaternion(
          cube.quaternion.clone().invert()
        );

        return .86 * (
          Math.abs(y.x) + Math.abs(y.y) + Math.abs(y.z)
        ) + .14;
      }

      function resize() {
        const width = host.clientWidth;
        const height = host.clientHeight;

        renderer.setSize(width, height, false);
        camera.aspect = width / height;
        camera.updateProjectionMatrix();

        const size = renderer.getDrawingBufferSize(
          new T.Vector2()
        );

        target.setSize(size.x, size.y);
        postMaterial.uniforms.resolution.value.copy(size);
        render();
      }

      const observer = new ResizeObserver(resize);
      observer.observe(host);

      // Interação pelo cursor, com inércia e retorno suave.
      const raycaster = new T.Raycaster();
      const pointer = new T.Vector2();
      const tablePlane = new T.Plane(
        new T.Vector3(0, 1, 0),
        -1
      );

      const contact = new T.Vector3();

      camera.updateMatrixWorld();

      const right = new T.Vector3().setFromMatrixColumn(
        camera.matrixWorld,
        0
      );

      right.y = 0;
      right.normalize();

      const forward = new T.Vector3(-right.z, 0, right.x);

      const motion = {
        x: 0,
        z: 0,
        tiltX: 0,
        tiltZ: 0,
        turn: 0,
        vx: 0,
        vz: 0,
        vtx: 0,
        vtz: 0,
        vt: 0
      };

      let restFace = lastFace || 1;
      let interactionFrame = null;
      let lastTime = 0;
      let previousPointer = null;
      let drag = null;
      let hover = false;

      const canvas = renderer.domElement;
      canvas.style.touchAction = 'pan-y';

      function stopInteraction() {
        if (interactionFrame !== null) {
          cancelAnimationFrame(interactionFrame);
        }

        interactionFrame = null;
        previousPointer = null;
        hover = false;

        if (drag && canvas.hasPointerCapture(drag.id)) {
          canvas.releasePointerCapture(drag.id);
        }

        drag = null;
        canvas.style.cursor = 'default';
        host.dataset.interacting = 'false';

        for (const key of Object.keys(motion)) {
          motion[key] = 0;
        }
      }

      function pointerRay(event) {
        const rect = canvas.getBoundingClientRect();

        pointer.set(
          (event.clientX - rect.left) / rect.width * 2 - 1,
          -(event.clientY - rect.top) / rect.height * 2 + 1
        );

        scene.updateMatrixWorld(true);
        camera.updateMatrixWorld();
        raycaster.setFromCamera(pointer, camera);

        return raycaster.intersectObject(cube, true).length > 0;
      }

      function applyMotion() {
        const tilt = new T.Quaternion().setFromEuler(
          new T.Euler(motion.tiltX, 0, motion.tiltZ)
        );

        cube.quaternion
          .copy(orientation(restFace, yaw + motion.turn))
          .premultiply(tilt);

        cube.position.set(
          motion.x,
          supportHeight(),
          motion.z
        );

        host.dataset.interacting = 'true';
        render();
      }

      function interactionTick(now) {
        if (animation || rolling) {
          stopInteraction();
          return;
        }

        const elapsed = Math.min((now - lastTime) / 1000, .25);
        lastTime = now;

        const steps = Math.max(1, Math.ceil(elapsed * 120));
        const dt = elapsed / steps;
        const damping = Math.exp(-7 * dt);

        const components = [
          ['x', 'vx'],
          ['z', 'vz'],
          ['tiltX', 'vtx'],
          ['tiltZ', 'vtz'],
          ['turn', 'vt']
        ];

        for (let step = 0; step < steps; step++) {
          for (const [position, velocity] of components) {
            if (drag && (position === 'x' || position === 'z')) {
              continue;
            }

            motion[velocity] = (
              motion[velocity] - motion[position] * 32 * dt
            ) * damping;

            motion[position] += motion[velocity] * dt;
          }
        }

        motion.x = T.MathUtils.clamp(motion.x, -1.3, 1.3);
        motion.z = T.MathUtils.clamp(motion.z, -1.1, 1.1);
        motion.tiltX = T.MathUtils.clamp(motion.tiltX, -.23, .23);
        motion.tiltZ = T.MathUtils.clamp(motion.tiltZ, -.23, .23);
        motion.turn = T.MathUtils.clamp(motion.turn, -.65, .65);

        applyMotion();

        const energy = Object.values(motion).reduce(
          (sum, value) => sum + Math.abs(value),
          0
        );

        if (drag || energy > .002) {
          interactionFrame = requestAnimationFrame(
            interactionTick
          );
        } else {
          const wasHover = hover;
          interactionFrame = null;
          show(restFace);
          canvas.style.cursor = wasHover ? 'grab' : 'default';
        }
      }

      function wakeInteraction() {
        if (interactionFrame === null) {
          lastTime = performance.now();
          interactionFrame = requestAnimationFrame(
            interactionTick
          );
        }
      }

      canvas.addEventListener('pointermove', event => {
        if (rolling || animation) return;

        const hit = pointerRay(event);
        canvas.style.cursor =
          drag ? 'grabbing' : hit ? 'grab' : 'default';

        if (drag) {
          if (event.pointerId !== drag.id) return;

          if (raycaster.ray.intersectPlane(tablePlane, contact)) {
            const x = T.MathUtils.clamp(
              contact.x - drag.offset.x, -1.3, 1.3
            );

            const z = T.MathUtils.clamp(
              contact.z - drag.offset.z, -1.1, 1.1
            );

            motion.vtx = T.MathUtils.clamp(
              motion.vtx + (z - motion.z) * 2, -2, 2
            );

            motion.vtz = T.MathUtils.clamp(
              motion.vtz - (x - motion.x) * 2, -2, 2
            );

            motion.x = x;
            motion.z = z;
            wakeInteraction();
          }

          return;
        }

        const previous = previousPointer;
        previousPointer = {
          x: event.clientX,
          y: event.clientY
        };

        const reducedMotion = window.matchMedia(
          '(prefers-reduced-motion: reduce)'
        ).matches;

        if (hit && event.pointerType !== 'touch' && !reducedMotion) {
          const dx = previous
            ? T.MathUtils.clamp(event.clientX - previous.x, -35, 35)
            : 3;

          const dy = previous
            ? T.MathUtils.clamp(event.clientY - previous.y, -35, 35)
            : 0;

          const push = right.clone()
            .multiplyScalar(dx)
            .addScaledVector(forward, dy)
            .multiplyScalar(.025);

          if (!hover) push.addScaledVector(right, .12);

          motion.vx = T.MathUtils.clamp(
            motion.vx + push.x, -3, 3
          );

          motion.vz = T.MathUtils.clamp(
            motion.vz + push.z, -3, 3
          );

          motion.vtx = T.MathUtils.clamp(
            motion.vtx + push.z * .8, -2, 2
          );

          motion.vtz = T.MathUtils.clamp(
            motion.vtz - push.x * .8, -2, 2
          );

          motion.vt = T.MathUtils.clamp(
            motion.vt + dx * .009, -2, 2
          );

          wakeInteraction();
        }

        hover = hit;
      });

      canvas.addEventListener('pointerdown', event => {
        if (
          rolling ||
          animation ||
          event.button !== 0 ||
          !pointerRay(event)
        ) {
          return;
        }

        if (!raycaster.ray.intersectPlane(tablePlane, contact)) {
          return;
        }

        drag = {
          id: event.pointerId,
          offset: new T.Vector3(
            contact.x - motion.x,
            0,
            contact.z - motion.z
          )
        };

        canvas.setPointerCapture(event.pointerId);
        canvas.style.cursor = 'grabbing';
        wakeInteraction();
      });

      function releasePointer(event) {
        if (!drag || event.pointerId !== drag.id) return;

        const id = drag.id;
        drag = null;

        if (canvas.hasPointerCapture(id)) {
          canvas.releasePointerCapture(id);
        }

        canvas.style.cursor = 'default';
        wakeInteraction();
      }

      canvas.addEventListener('pointerup', releasePointer);
      canvas.addEventListener('pointercancel', releasePointer);
      canvas.addEventListener('lostpointercapture', releasePointer);

      canvas.addEventListener('pointerleave', () => {
        previousPointer = null;
        hover = false;

        if (!drag) canvas.style.cursor = 'default';
      });

      window.addEventListener('blur', () => {
        if (drag) releasePointer({ pointerId: drag.id });
        previousPointer = null;
      });

      function show(face) {
        stopInteraction();
        restFace = face;
        animation = null;

        if (frame !== null) cancelAnimationFrame(frame);
        frame = null;

        cube.quaternion.copy(orientation(face, yaw));
        cube.position.set(0, 1, 0);

        const topIndex = normals.reduce((best, normal, index) => {
          const height = normal.clone()
            .applyQuaternion(cube.quaternion).y;

          const bestHeight = normals[best].clone()
            .applyQuaternion(cube.quaternion).y;

          return height > bestHeight ? index : best;
        }, 0);

        host.dataset.topFace = String(topIndex + 1);
        render();
      }

      function roll(face, duration) {
        stopInteraction();

        if (!duration) {
          show(face);
          return;
        }

        const start = performance.now();
        const initial = cube.quaternion.clone();
        yaw = (Math.random() - .5) * 1.1;

        const destination = orientation(face, yaw);

        const axis = new T.Vector3(
          Math.random() + .3,
          Math.random() + .2,
          Math.random() + .3
        ).normalize();

        const turns =
          (3 + Math.floor(Math.random() * 3)) * Math.PI * 2;

        const direction = Math.random() > .5 ? 1 : -1;
        const travel = (Math.random() - .5) * .75;

        animation = { face };
        host.dataset.topFace = 'rolling';

        function tick(now) {
          if (!animation) return;

          const t = Math.min(1, (now - start) / duration);
          const ease = 1 - Math.pow(1 - t, 3);

          cube.quaternion.copy(initial).slerp(destination, ease);

          const spin = new T.Quaternion().setFromAxisAngle(
            axis,
            direction * turns * Math.pow(1 - t, 3)
          );

          cube.quaternion.premultiply(spin);

          let jump = 0;

          if (t < .5) {
            jump = Math.sin(Math.PI * t / .5) * 1.65;
          } else if (t < .76) {
            jump = Math.sin(Math.PI * (t - .5) / .26) * .38;
          } else {
            jump = Math.sin(Math.PI * (t - .76) / .24) * .07;
          }

          cube.position.set(
            Math.sin(t * Math.PI) * travel,
            supportHeight() + jump,
            Math.sin(t * Math.PI) * travel * .5
          );

          render();

          if (t < 1) {
            frame = requestAnimationFrame(tick);
          } else {
            show(face);
          }
        }

        frame = requestAnimationFrame(tick);
      }

      canvas.addEventListener('webglcontextlost', event => {
        event.preventDefault();
        stopInteraction();
        animation = null;

        if (frame !== null) cancelAnimationFrame(frame);

        observer.disconnect();
        diceView = null;
        host.classList.remove('ready');
        canvas.remove();
        drawFace(die, lastFace || 1);

        document.getElementById('scene-note').textContent =
          'A renderização 3D foi interrompida. ' +
          'O modo simples continua disponível; ' +
          'recarregue a página para tentar recuperar o 3D.';
      });

      host.classList.add('ready');
      resize();
      show(lastFace || 1);

      return { show, roll };
    }

    try {
      diceView = createDiceView();
    } catch {
      document.getElementById('scene')
        .querySelector('canvas')?.remove();

      document.getElementById('scene-note').textContent =
        'Visualização simples: o 3D precisa de internet e de ' +
        'um navegador com WebGL. Os lançamentos e as ' +
        'estatísticas continuam disponíveis.';
    }

    function randomFace() {
      if (
        globalThis.crypto &&
        typeof globalThis.crypto.getRandomValues === 'function'
      ) {
        const value = new Uint32Array(1);

        do {
          globalThis.crypto.getRandomValues(value);
        } while (value[0] >= 4294967292);

        return value[0] % 6 + 1;
      }

      return Math.floor(Math.random() * 6) + 1;
    }

    rollButton.addEventListener('click', () => {
      if (rolling) return;

      const face = randomFace();

      rolling = true;
      rollButton.disabled = true;
      resetButton.disabled = true;
      rollButton.textContent = 'Jogando…';
      result.textContent = 'O dado está rolando…';

      if (!diceView) die.classList.add('rolling');

      const reducedMotion = window.matchMedia(
        '(prefers-reduced-motion: reduce)'
      ).matches;

      const duration =
        reducedMotion ? 0 : (diceView ? 1800 : 700);

      if (diceView) diceView.roll(face, duration);

      const interval = (reducedMotion || diceView)
        ? null
        : window.setInterval(() => {
            drawFace(die, Math.floor(Math.random() * 6) + 1);
          }, 90);

      window.setTimeout(() => {
        if (interval !== null) window.clearInterval(interval);

        die.classList.remove('rolling');
        drawFace(die, face);

        counts[face - 1]++;
        lastFace = face;

        updateStats();
        saveStats();

        result.textContent = 'Você tirou a face ' + face + '.';

        rolling = false;
        rollButton.disabled = false;
        resetButton.disabled = false;
        rollButton.textContent = 'Jogar dado';
      }, duration);
    });

    resetButton.addEventListener('click', () => {
      if (rolling) return;

      counts = [0, 0, 0, 0, 0, 0];
      lastFace = null;

      updateStats();
      saveStats();
      drawFace(die, 1);

      result.textContent =
        'Estatísticas reiniciadas. Faça um novo lançamento.';
    });
  </script>
</body>
</html>

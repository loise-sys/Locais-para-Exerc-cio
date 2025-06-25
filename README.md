<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Locais para Exercício</title>
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap" rel="stylesheet" />
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
<style>
  :root {
    --primary: #48c9b0;
    --secondary: #5dade2;
    --bg-light: #eef8f5;
    --bg-dark: #1e272e;
    --text-light: #ffffff;
    --text-dark: #2c3e50;
    --card-bg-light: #fff;
    --card-bg-dark: #2d3436;
    --sidebar-width: 300px;
  }

  [data-theme="dark"] {
    background-color: var(--bg-dark);
    color: var(--text-light);
  }
  [data-theme="dark"] header,
  [data-theme="dark"] #sidebar {
    background: linear-gradient(90deg, #26a69a, #2980b9);
  }
  [data-theme="dark"] button,
  [data-theme="dark"] input {
    background: linear-gradient(to right, #2980b9, #26a69a);
    color: #fff;
    border-color: transparent;
  }
  [data-theme="dark"] .leaflet-popup-content {
    background-color: var(--card-bg-dark);
    color: #fff;
  }

  * {
    box-sizing: border-box;
    margin: 0; padding: 0;
  }

  body {
    font-family: 'Poppins', sans-serif;
    background-color: var(--bg-light);
    color: var(--text-dark);
    display: flex;
    height: 100vh;
    overflow: hidden;
    transition: background-color 0.3s, color 0.3s;
  }

  #sidebar {
    width: var(--sidebar-width);
    background: linear-gradient(90deg, var(--primary), var(--secondary));
    padding: 2rem 1.5rem;
    color: white;
    display: flex;
    flex-direction: column;
    box-shadow: 2px 0 8px rgba(0,0,0,0.1);
    transition: transform 0.3s ease;
    z-index: 1000;
  }

  #sidebar.collapsed {
    transform: translateX(-100%);
  }

  #sidebar h1 {
    font-size: 1.8rem;
    margin-bottom: 1rem;
  }

  #sidebar p {
    opacity: 0.9;
    margin-bottom: 2rem;
    font-size: 1rem;
  }

  #toggle-sidebar-btn {
    display: none;
    position: fixed;
    top: 10px; left: 10px;
    background: var(--primary);
    border: none;
    color: white;
    padding: 10px 15px;
    font-size: 1.2rem;
    border-radius: 8px;
    cursor: pointer;
    z-index: 1100;
    box-shadow: 0 2px 6px rgba(0,0,0,0.25);
  }

  #filters button,
  #toggle-theme,
  #clear-route-btn {
    margin: 0.4rem 0;
    padding: 0.7rem 1.4rem;
    border: none;
    border-radius: 25px;
    background: linear-gradient(to right, var(--secondary), var(--primary));
    color: white;
    font-weight: 600;
    cursor: pointer;
    font-size: 1rem;
    box-shadow: 0 3px 6px rgba(0,0,0,0.15);
    transition: all 0.3s ease;
    width: 100%;
  }

  #filters button:hover,
  #toggle-theme:hover,
  #clear-route-btn:hover,
  #filters button:focus,
  #toggle-theme:focus,
  #clear-route-btn:focus {
    transform: translateY(-2px);
    outline: 2px solid #fff;
  }

  #search-container input {
    padding: 0.7rem 1rem;
    border-radius: 20px;
    border: 1px solid #ccc;
    font-size: 1rem;
    width: 100%;
    margin: 1rem 0 2rem 0;
  }

  #map {
    flex-grow: 1;
    height: 100vh;
    width: 100%;
    transition: margin-left 0.3s ease;
  }

  [data-theme="dark"] #map {
    background-color: var(--bg-dark);
  }

  .leaflet-popup-content img {
    width: 100%;
    max-height: 160px;
    object-fit: cover;
    border-radius: 10px;
    margin-top: 8px;
    box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  }

  .emoji-icon {
    text-align: center;
    line-height: 30px;
    font-size: 24px;
    transition: transform 0.3s ease;
  }

  .emoji-icon:hover {
    transform: scale(1.2);
  }

  @media (max-width: 768px) {
    #sidebar {
      position: fixed;
      height: 100vh;
      top: 0;
      left: 0;
      z-index: 1000;
    }

    #toggle-sidebar-btn {
      display: block;
    }

    #map {
      margin-left: 0 !important;
    }
  }
</style>
</head>
<body>

<button id="toggle-sidebar-btn" aria-label="Abrir/fechar menu lateral">☰</button>

<div id="sidebar" role="complementary" aria-label="Menu lateral de filtros e controles">
  <h1>🌿 Locais para Exercício</h1>
  <p>Descubra espaços perto de você para se movimentar com saúde!</p>

  <div id="filters" aria-label="Filtros de locais para exercício">
    <button onclick="filtrar('todos')" aria-label="Mostrar todos os locais">Todos</button>
    <button onclick="filtrar('caminhada')" aria-label="Locais de caminhada">🚶 Caminhada</button>
    <button onclick="filtrar('musculacao')" aria-label="Locais de musculação">🏋️ Musculação</button>
    <button onclick="filtrar('yoga')" aria-label="Locais de yoga">🧘 Yoga</button>
  </div>

  <div id="search-container">
    <input type="text" id="search" placeholder="Pesquisar local..." aria-label="Pesquisar local" oninput="pesquisarLocal()" />
  </div>

  <button id="toggle-theme" onclick="alternarTema()" aria-label="Alternar tema claro e escuro">🌙 Alternar Tema</button>
  <button id="clear-route-btn" onclick="limparRota()" aria-label="Limpar rota do mapa" style="margin-top:1rem;">🧹 Limpar Rota</button>
</div>

<div id="map" role="region" aria-label="Mapa com locais de exercício"></div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<script>
  const MAPBOX_TOKEN = 'pk.eyJ1IjoibWFwYm94LXVzZXIiLCJhIjoiY2t2NGR2cGltMDAxczJ2bGowbnY5N3ZqMyJ9.Yo2dtGTolLQhRkGZ42wPQA'; // substitua pelo seu token válido

  let temaAtual = localStorage.getItem('tema') || 'light';
  document.body.setAttribute('data-theme', temaAtual);

  function alternarTema() {
    temaAtual = temaAtual === 'light' ? 'dark' : 'light';
    document.body.setAttribute('data-theme', temaAtual);
    localStorage.setItem('tema', temaAtual);
  }

  const sidebar = document.getElementById('sidebar');
  const toggleSidebarBtn = document.getElementById('toggle-sidebar-btn');
  toggleSidebarBtn.addEventListener('click', () => {
    sidebar.classList.toggle('collapsed');
  });

  if(window.innerWidth <= 768){
    sidebar.classList.add('collapsed');
  }
  window.addEventListener('resize', () => {
    if(window.innerWidth > 768){
      sidebar.classList.remove('collapsed');
    } else {
      sidebar.classList.add('collapsed');
    }
  });

  const map = L.map('map').setView([-15.5709215, -56.0689904], 13);
  let rotaPolyline = null;
  const marcadores = [];

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(map);

  const icons = {
    caminhada: L.divIcon({ className: 'emoji-icon', html: '🚶‍♀️', iconSize: [30, 30], popupAnchor: [0, -15] }),
    musculacao: L.divIcon({ className: 'emoji-icon', html: '🏋️‍♂️', iconSize: [30, 30], popupAnchor: [0, -15] }),
    yoga: L.divIcon({ className: 'emoji-icon', html: '🧘‍♀️', iconSize: [30, 30], popupAnchor: [0, -15] })
  };

const locais = [
  {
    nome: "Praça Tenente Luciano Maiolino (Bela Vista)",
    descricao: "Pista de caminhada e academia ao ar livre.",
    imagem: "https://images.openai.com/thumbnails/url/SPXUKHicu1mUUVJSUGylr5-al1xUWVCSmqJbkpRnoJdeXJJYkpmsl5yfq5-Zm5ieWmxfaAuUsXL0S7F0Tw5JLQ8oCzczLg0PKnQ2rQjJdcxLTa7ycU928wwMt3AuiCpJd_dLC3RKSrdMDTZ2KvUJywwxijROzDVTKwYA0acpFg",
    tipo: "caminhada",
    lat: -15.5823723,
    lng: -56.0596596
  },
  {
    nome: "Praça Gonçalo Miranda",
    descricao: "Caminhada e equipamentos de musculação.",
    imagem: "https://images.openai.com/thumbnails/url/nCq-zHicu1mUUVJSUGylr5-al1xUWVCSmqJbkpRnoJdeXJJYkpmsl5yfq5-Zm5ieWmxfaAuUsXL0S7F0Tw52yzEPSQn1LKkyKvWMqMrwDTBLiYx0NTfJLfNyda5MN0wtCfLOMza08CnLLnMuC8_ySjUNTDeNKE5WKwYAxWwpNA",
    tipo: "musculacao",
    lat: -15.5843181,
    lng: -56.0611468
  },
  {
    nome: "Praça do Terra Nova",
    descricao: "Pista de caminhada, academia ao ar livre, playground e iluminação LED.",
    imagem: "https://images.openai.com/thumbnails/url/5WylR3icu1mSUVJSUGylr5-al1xUWVCSmqJbkpRnoJdeXJJYkpmsl5yfq5-Zm5ieWmxfaAuUsXL0S7F0Tw5OM8lwKggs9vOwCPINSAvMNg31yY_ILHN3NvfICEqJKnIOTopItfTMKHdNjD193YPzHPyDC70qkg2K1crBgATlCnK",
    tipo: "musculacao",
    lat: -15.5812044,
    lng: -56.0713768
  },
  {
    nome: "Praça do Amarelinho",
    descricao: "Local arborizado para caminhadas.",
    imagem: "https://images.openai.com/thumbnails/url/1424qHicu1mSUVJSUGylr5-al1xUWVCSmqJbkpRnoJdeXJJYkpmsl5yfq5-Zm5ieWmxfaAuUsXL0S7F0Tw5y8srz98s0MswISXEpDnMMNQ8KCEoyjnd28imNL8jSDXJ0L68MTKysKMsOD3YMKswz8zb1yHYxKk4MS1crBgAJQimu",
    tipo: "caminhada",
    lat: -15.5694683,
    lng: -56.061793
  },
  {
    nome: "Praça Morada do Ouro",
    descricao: "Espaço tranquilo para caminhada.",
    imagem: "https://images.openai.com/thumbnails/url/5WylR3icu1mSUVJSUGylr5-al1xUWVCSmqJbkpRnoJdeXJJYkpmsl5yfq5-Zm5ieWmxfaAuUsXL0S7F0Tw5OM8lwKggs9vOwCPINSAvMNg31yY_ILHN3NvfICEqJKnIOTopItfTMKHdNjD193YPzHPyDC70qkg2K1crBgATlCnK",
    tipo: "caminhada",
    lat: -15.5661659,
    lng: -56.0600054
  },
  {
    nome: "Praça Umberlindo Silva Nascimento",
    descricao: "Caminhada, musculação e espaço para exercícios.",
    imagem: "https://images.openai.com/thumbnails/url/nCq-zHicu1mUUVJSUGylr5-al1xUWVCSmqJbkpRnoJdeXJJYkpmsl5yfq5-Zm5ieWmxfaAuUsXL0S7F0Tw52yzEPSQn1LKkyKvWMqMrwDTBLiYx0NTfJLfNyda5MN0wtCfLOMza08CnLLnMuC8_ySjUNTDeNKE5WKwYAxWwpNA",
    tipo: "musculacao",
    lat: -15.588043,
    lng: -56.0554175
  },
  {
    nome: "Ginásio Verdinho",
    descricao: "Ginásio com quadras e espaço para treino funcional.",
    imagem: "https://images.openai.com/thumbnails/url/5WylR3icu1mSUVJSUGylr5-al1xUWVCSmqJbkpRnoJdeXJJYkpmsl5yfq5-Zm5ieWmxfaAuUsXL0S7F0Tw5OM8lwKggs9vOwCPINSAvMNg31yY_ILHN3NvfICEqJKnIOTopItfTMKHdNjD193YPzHPyDC70qkg2K1crBgATlCnK",
    tipo: "musculacao",
    lat: -15.5538904,
    lng: -56.0561104
  },
  {
    nome: "Parque da Nascente",
    descricao: "Área verde preservada para caminhada.",
    imagem: "https://images.openai.com/thumbnails/url/5WylR3icu1mSUVJSUGylr5-al1xUWVCSmqJbkpRnoJdeXJJYkpmsl5yfq5-Zm5ieWmxfaAuUsXL0S7F0Tw5OM8lwKggs9vOwCPINSAvMNg31yY_ILHN3NvfICEqJKnIOTopItfTMKHdNjD193YPzHPyDC70qkg2K1crBgATlCnK",
    tipo: "caminhada",
    lat: -15.5719303,
    lng: -56.0570296
  },
  {
    nome: "Parque Municipal Lagoa Encantada",
    descricao: "Ambiente arborizado para exercícios, caminhada e relaxamento.",
    imagem: "https://images.openai.com/thumbnails/url/5WylR3icu1mSUVJSUGylr5-al1xUWVCSmqJbkpRnoJdeXJJYkpmsl5yfq5-Zm5ieWmxfaAuUsXL0S7F0Tw5OM8lwKggs9vOwCPINSAvMNg31yY_ILHN3NvfICEqJKnIOTopItfTMKHdNjD193YPzHPyDC70qkg2K1crBgATlCnK",
    tipo: "musculação",
    lat: -15.5675453,
    lng: -56.0408212
  },
  {
  nome: "VÔLEI BELA VISTA",
  descricao: "Quadra de areia para vôlei, exercícios e caminhada.",
  imagem: "https://via.placeholder.com/300x150.png?text=V%C3%B4lei+Bela+Vista",
  tipo: "musculacao",
  lat: -15.5844449,
  lng: -56.0609833
},
{
  nome: "Campo de Areia",
  descricao: "Espaço para exercícios funcionais e caminhadas.",
  imagem: "https://via.placeholder.com/300x150.png?text=Campo+de+Areia",
  tipo: "caminhada",
  lat: -15.5725515,
  lng: -56.0621832
},
{
  nome: "BASQUETE CPA",
  descricao: "Quadra de basquete usada para treinos e exercícios ao ar livre.",
  imagem: "https://via.placeholder.com/300x150.png?text=Basquete+CPA",
  tipo: "exercicios",
  lat: -15.556539,
  lng: -56.0514005
},
{
  nome: "Cena Society CPA",
  descricao: "Quadra society para atividades físicas diversas.",
  imagem: "https://via.placeholder.com/300x150.png?text=Cena+Society+CPA",
  tipo: "exercicios",
  lat: -15.5586669,
  lng: -56.0492886
},
{
  nome: "Mini Estádio CPA 3 Setor 4",
  descricao: "Campo com espaço para musculação e caminhada.",
  imagem: "https://via.placeholder.com/300x150.png?text=CPA+3+Setor+4",
  tipo: "musculacao",
  lat: -15.5710831,
  lng: -56.0407223
},
{
  nome: "Centro de Treinamento do Clube Atlético Três Lagoas",
  descricao: "Centro esportivo com estrutura para treino e exercícios.",
  imagem: "https://via.placeholder.com/300x150.png?text=Tr%C3%AAs+Lagoas",
  tipo: "musculacao",
  lat: -15.5648054,
  lng: -56.0401051
},
{
  nome: "Arena Conquista",
  descricao: "Área multiuso para caminhada e treinos físicos.",
  imagem: "https://via.placeholder.com/300x150.png?text=Arena+Conquista",
  tipo: "musculacao",
  lat: -15.5513918,
  lng: -56.0385652
},
{
  nome: "Arenas da Serra",
  descricao: "Espaço esportivo com estrutura para musculação e atividades em grupo.",
  imagem: "https://via.placeholder.com/300x150.png?text=Arenas+da+Serra",
  tipo: "musculacao",
  lat: -15.5516023,
  lng: -56.0388234
}
];

  function adicionarMarcadores(tipoFiltro) {
    marcadores.forEach(m => map.removeLayer(m));
    marcadores.length = 0;

    locais.forEach(local => {
      if (tipoFiltro === "todos" || local.tipo === tipoFiltro) {
        const marker = L.marker([local.lat, local.lng], { icon: icons[local.tipo] || icons["caminhada"] }).addTo(map);
        marker._icon.style.opacity = 0;
        marker._icon.style.transform = "scale(0.5)";
        setTimeout(() => {
          marker._icon.style.transition = "all 0.4s ease";
          marker._icon.style.opacity = 1;
          marker._icon.style.transform = "scale(1)";
        }, 10);

        marker.bindPopup(`
          <strong>${local.nome}</strong><br>
          ${local.descricao}<br>
          <img src="${local.imagem}" alt="Imagem de ${local.nome}" />
          <br><button onclick="tracarRota(${local.lat}, ${local.lng})">Ver Rota</button>
        `);
        marcadores.push(marker);
      }
    });
  }

  function filtrar(tipo) {
    adicionarMarcadores(tipo);
  }

  function pesquisarLocal() {
    const termo = document.getElementById('search').value.toLowerCase();
    const filtrados = locais.filter(local => local.nome.toLowerCase().includes(termo));
    marcadores.forEach(m => map.removeLayer(m));
    marcadores.length = 0;

    filtrados.forEach(local => {
      const marker = L.marker([local.lat, local.lng], { icon: icons[local.tipo] || icons["caminhada"] }).addTo(map);
      marker.bindPopup(`
        <strong>${local.nome}</strong><br>
        ${local.descricao}<br>
        <img src="${local.imagem}" alt="Imagem de ${local.nome}" />
        <br><button onclick="tracarRota(${local.lat}, ${local.lng})">Ver Rota</button>
      `);
      marcadores.push(marker);
    });
  }

  async function tracarRota(destLat, destLng) {
    if (rotaPolyline) {
      map.removeLayer(rotaPolyline);
      rotaPolyline = null;
    }

    if (!navigator.geolocation) {
      alert('Geolocalização não suportada no seu navegador.');
      return;
    }

    navigator.geolocation.getCurrentPosition(async (position) => {
      const userLat = position.coords.latitude;
      const userLng = position.coords.longitude;

      const url = `https://api.mapbox.com/directions/v5/mapbox/walking/${userLng},${userLat};${destLng},${destLat}?geometries=geojson&access_token=${MAPBOX_TOKEN}`;

      try {
        const resp = await fetch(url);
        if (!resp.ok) throw new Error('Erro na API de rotas');
        const data = await resp.json();
        const coords = data.routes[0].geometry.coordinates.map(c => [c[1], c[0]]);

        rotaPolyline = L.polyline(coords, { color: 'red', weight: 5 }).addTo(map);
        map.fitBounds(rotaPolyline.getBounds(), { padding: [50, 50] });

      } catch (error) {
        alert('Erro ao buscar rota. Tente novamente mais tarde.');
        console.error(error);
      }
    }, () => {
      alert('Não foi possível acessar sua localização.');
    });
  }

  function limparRota() {
    if (rotaPolyline) {
      map.removeLayer(rotaPolyline);
      rotaPolyline = null;
    }
  }

  adicionarMarcadores("todos");
</script>

</body>
</html>

<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mapa Local Inteligente</title>
<link rel="icon" href="icono.png">
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<meta name="theme-color" content="#4285F4">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="mobile-web-app-capable" content="yes">
<style>
body{
    margin:0;
    font-family:Arial,sans-serif;
}

#map{
    width:100%;
    height:70vh;
}

#info{
    padding:15px;
    background:#f5f5f5;
}

.card{
  
    background:white;
    padding:10px;
    margin:10px 0;
    border-radius:10px;
    box-shadow:0 2px 5px rgba(0,0,0,.2);
}
  #herramientas{
    position:absolute;
    top:55px;
    left:50px;
    z-index:1000;
    display:flex;
    gap:5px;
    flex-wrap:wrap;
    max-width:600px;
}

#herramientas button{
    border:none;
    padding:8px 12px;
    border-radius:20px;
    background:white;
    cursor:pointer;
    box-shadow:0 2px 5px rgba(0,0,0,.3);
  
}

.leaflet-control-geocoder{
    width:300px !important;
}

.leaflet-control-geocoder-form input{
    font-size:16px !important;
    padding:8px !important;
}

</style>
</head>
<body>
<div id="loginPanel" style="
position:absolute;
top:10px;
right:10px;
z-index:9999;
background:white;
padding:10px;
border-radius:10px;
box-shadow:0 2px 5px rgba(0,0,0,.3);
">
<button onclick="login()">🔑 Iniciar sesión</button>
<button onclick="logout()">🚪 Salir</button>
<div id="usuario">No has iniciado sesión</div>
</div>

<div id="map"></div>

<input
id="buscar"
type="text"
placeholder="Buscar lugar..."
style="
position:absolute;
top:10px;
left:50px;
z-index:1000;
padding:8px;
width:250px;">
  <div id="herramientas">

<button id="btnRestaurantes">🍔 Restaurantes</button>
<button id="btnGasolineras">⛽ Gasolineras</button>
<button id="btnHospitales">🏥 Hospitales</button>
<button id="btnEscuelas">🏫 Escuelas</button>
<button id="btnHoteles">🏨 Hoteles</button>
<button id="btnTiendas">🛒 Tiendas</button>
<button id="btnTurismo">🏖️ Turismo</button>
<button id="btnClima">🌦️ Clima</button>
<button id="guardarFavorito">⭐ Guardar favorito</button>
</div>
<div id="info">

  <div class="card">
    <h2>📍 Ubicación</h2>
    <p id="ubicacion">Coatzacoalcos, Veracruz, México</p>

    <p id="coordenadas">
        🌎 Latitud: 18.076180<br>
        🧭 Longitud: -94.476364
    </p>
</div>

<div class="card">
    <h2>🌤️ Clima</h2>
    <p id="clima">⏳ Cargando...</p>
</div>

<div class="card">
    <h2>⭐ Lugares favoritos</h2>
    <ul id="listaFavoritos"></ul>
</div>
  
    <div class="card">
        <h2>🍽️ Comida típica</h2>
        <ul>
            <li>Minilla de pescado</li>
            <li>Arroz a la tumbada</li>
            <li>Robalo al mojo de ajo</li>
            <li>Tamales de elote</li>
            <li>Pescado relleno de mariscos</li>
            <li>Vuelve a la vida</li>
          <li>Torito</li>
        </ul>
    </div>

</div>

<link rel="stylesheet"
href="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.css"/>
  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>

<script>


const supabaseClient = window.supabase.createClient(
    "https://mfumgaqdgsiemzynhebe.supabase.co",
    "sb_publishable_jhYBAxoRwKN6qxy-2hSCpQ_FEzDmwf3"
);
  
const lat = 18.07618;
const lon = -94.4763644;

// Crear mapa
const map = L.map('map').setView([lat, lon], 18);
let marcador = L.marker([lat, lon]).addTo(map);

const iconoColegio = L.icon({
    iconUrl: "icono.png", // tu imagen
    iconSize: [45, 45],
    iconAnchor: [22, 45],
    popupAnchor: [0, -40]
});
  // Buscador tipo Google Maps
L.Control.geocoder({
    defaultMarkGeocode: false
})
  
.on('markgeocode', function(e){

    const centro = e.geocode.center;

    map.setView(centro, 16);

    marcador.setLatLng(centro);

    marcador.bindPopup(
        e.geocode.name
    ).openPopup();

    document.getElementById("ubicacion").textContent =
        e.geocode.name;
ubicacionActual = {
    nombre: e.geocode.name,
    lat: centro.lat,
    lon: centro.lng
};
    document.getElementById("coordenadas").innerHTML =
        `🌎 Latitud: ${centro.lat.toFixed(6)}<br>
         🧭 Longitud: ${centro.lng.toFixed(6)}`;

    actualizarClima(
        centro.lat,
        centro.lng
    );

})
.addTo(map);
marcador.bindPopup("Ubicación seleccionada").openPopup();

map.on('click', function(e){
document.getElementById("ubicacion").textContent =
"Ubicación seleccionada";
    const nuevaLat = e.latlng.lat;
    const nuevaLon = e.latlng.lng;
ubicacionActual = {
    nombre: "Ubicación seleccionada",
    lat: nuevaLat,
    lon: nuevaLon
};
    marcador.setLatLng([nuevaLat, nuevaLon]);
document.getElementById("coordenadas").innerHTML =
`Latitud: ${nuevaLat.toFixed(6)}<br>
Longitud: ${nuevaLon.toFixed(6)}`;
    marcador.bindPopup(
        `Latitud: ${nuevaLat.toFixed(6)}<br>
         Longitud: ${nuevaLon.toFixed(6)}`
    ).openPopup();
actualizarClima(nuevaLat, nuevaLon);
});
// Capa satelital
const satelite = L.tileLayer(
'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}'
);

const calles = L.tileLayer(
'https://tile.openstreetmap.org/{z}/{x}/{y}.png'
);

satelite.addTo(map);

L.control.layers({
    "🗺️ Mapa": calles,
    "🛰️ Satélite": satelite
}).addTo(map);

async function actualizarClima(latitud, longitud){

    const clima = document.getElementById("clima");
    clima.innerHTML = "⏳ Cargando clima...";

    try{

        const respuesta = await fetch(
            `https://api.open-meteo.com/v1/forecast?latitude=${latitud}&longitude=${longitud}&current=temperature_2m,relative_humidity_2m,weather_code`
        );

        const data = await respuesta.json();

        let estado = "☀️ Despejado";

        switch(data.current.weather_code){

            case 0:
                estado = "☀️ Despejado";
                break;

            case 1:
            case 2:
            case 3:
                estado = "⛅ Parcialmente nublado";
                break;

            case 45:
            case 48:
                estado = "🌫️ Niebla";
                break;

            case 51:
            case 53:
            case 55:
                estado = "🌦️ Llovizna";
                break;

            case 61:
            case 63:
            case 65:
                estado = "🌧️ Lluvia";
                break;

            case 71:
            case 73:
            case 75:
                estado = "❄️ Nieve";
                break;

            case 95:
                estado = "⛈️ Tormenta";
                break;

            default:
                estado = "🌤️ Clima desconocido";
        }

        clima.innerHTML = `
            <b>${estado}</b><br>
            🌡️ Temperatura: ${data.current.temperature_2m} °C<br>
            💧 Humedad: ${data.current.relative_humidity_2m}%`;

    }catch(e){

        clima.innerHTML = "❌ No se pudo cargar el clima.";

    }

}
// Cargar clima inicial
actualizarClima(lat, lon);
  
  // Buscador de lugares
  const cajaBusqueda = document.getElementById("buscar");

cajaBusqueda.addEventListener("keypress", async function(e){

    if(e.key !== "Enter") return;

    const texto = cajaBusqueda.value;
if (
    texto.toLowerCase() === "colegio teresita" ||
    texto.toLowerCase() === "colegio teresita minatitlán a.c."
){

    const latColegio = 17.992615;
    const lonColegio = -94.545079;

    map.setView([latColegio, lonColegio], 19);

  map.removeLayer(marcador);

marcador = L.marker([latColegio, lonColegio], {
    icon: iconoColegio
}).addTo(map);

    marcador.bindPopup("<b>Colegio Teresita</b>").openPopup();

    document.getElementById("ubicacion").textContent =
        "Colegio Teresita";

    document.getElementById("coordenadas").innerHTML =
        `🌎 Latitud: ${latColegio}<br>
         🧭 Longitud: ${lonColegio}`;

    ubicacionActual = {
        nombre: "Colegio Teresita",
        lat: latColegio,
        lon: lonColegio
    };

    actualizarClima(latColegio, lonColegio);

    return;
}
    const resultado = await fetch(
        `https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(texto)}`
    );

    const lugares = await resultado.json();

    if(lugares.length === 0){
        alert("Lugar no encontrado");
        return;
    }

const lugar = lugares[0];

const lat = parseFloat(lugar.lat);
const lon = parseFloat(lugar.lon);

map.setView([lat, lon], 16);

if (map.hasLayer(marcador)) {
    map.removeLayer(marcador);
}

marcador = L.marker([lat, lon]).addTo(map);

marcador.bindPopup(lugar.display_name).openPopup();
    document.getElementById("ubicacion").textContent =
        lugar.display_name;
ubicacionActual = {
    nombre: lugar.display_name,
    lat: lat,
    lon: lon
};
    document.getElementById("coordenadas").innerHTML =
        `🌎 Latitud: ${lat.toFixed(6)}<br>
         🧭 Longitud: ${lon.toFixed(6)}`;

    actualizarClima(lat, lon);

});
  
  // =====================
// FAVORITOS
// =====================

let ubicacionActual = {
    nombre: "Coatzacoalcos, Veracruz",
    lat: lat,
    lon: lon
};

function mostrarFavoritos(){

    const lista = document.getElementById("listaFavoritos");
    lista.innerHTML = "";

    const favoritos =
        JSON.parse(localStorage.getItem("favoritos")) || [];

favoritos.forEach((fav, indice)=>{

    const li = document.createElement("li");

    li.innerHTML =
    `<a href="#" data-i="${indice}">
        ${fav.nombre}
    </a>`;

    lista.appendChild(li);

li.querySelector("a").onclick = function(e){
    e.preventDefault();

        ubicacionActual = fav;

        document.getElementById("ubicacion").textContent =
            fav.nombre;

        document.getElementById("coordenadas").innerHTML =
        `🌎 Latitud: ${fav.lat.toFixed(6)}<br>
         🧭 Longitud: ${fav.lon.toFixed(6)}`;

        map.setView([fav.lat, fav.lon], 16);

if (map.hasLayer(marcador)) {
    map.removeLayer(marcador);
}

marcador = L.marker([fav.lat, fav.lon]).addTo(map);

marcador.bindPopup(fav.nombre).openPopup();

        actualizarClima(fav.lat, fav.lon);

    };

});

}

document.getElementById("guardarFavorito")
.addEventListener("click", ()=>{

    const favoritos =
        JSON.parse(localStorage.getItem("favoritos")) || [];

    favoritos.push(ubicacionActual);

    localStorage.setItem(
        "favoritos",
        JSON.stringify(favoritos)
    );

    mostrarFavoritos();

    alert("⭐ Lugar guardado");

});

mostrarFavoritos();
  let marcadoresBusqueda = [];

function limpiarBusqueda(){
    marcadoresBusqueda.forEach(m => map.removeLayer(m));
    marcadoresBusqueda = [];
}

async function buscarLugares(tipo){

    limpiarBusqueda();

    const centro = map.getCenter();
    const lat = centro.lat;
    const lon = centro.lng;

    let consulta = "";

if(tipo === "restaurant")
consulta = `
node["amenity"~"restaurant|fast_food|cafe"](around:5000,${lat},${lon});
way["amenity"~"restaurant|fast_food|cafe"](around:5000,${lat},${lon});
relation["amenity"~"restaurant|fast_food|cafe"](around:5000,${lat},${lon});
`;
    if(tipo === "fuel")
        consulta = `node["amenity"="fuel"](around:3000,${lat},${lon});`;

    if(tipo === "hospital")
        consulta = `node["amenity"="hospital"](around:3000,${lat},${lon});`;

    if(tipo === "school")
        consulta = `node["amenity"="school"](around:3000,${lat},${lon});`;
    if(tipo === "hotel")
        consulta = `node["tourism"="hotel"](around:3000,${lat},${lon});`;

    if(tipo === "shop")
        consulta = `node["shop"](around:3000,${lat},${lon});`;

    if(tipo === "tourism")
        consulta = `node["tourism"](around:3000,${lat},${lon});`;
    const query = `
        [out:json];
        (
            ${consulta}
        );
        out center;
    `;

  let data;

try{
const response = await fetch(
    "https://overpass-api.de/api/interpreter",
    {
        method: "POST",
        body: query
    }
);

const texto = await response.text();
console.log(texto);

data = JSON.parse(texto);
  console.log(data);
  console.log(query);
console.log(data.elements);
}catch(error){
    console.error(error);
    alert("Error buscando lugares: " + error.message);
    return;
}
if (!data.elements || data.elements.length === 0) {
    alert("No se encontraron lugares");
    return;
}

data.elements.forEach(lugar => {

    const latitud = lugar.lat || (lugar.center && lugar.center.lat);
    const longitud = lugar.lon || (lugar.center && lugar.center.lon);

    if (!latitud || !longitud) return;

    const marker = L.marker([latitud, longitud]).addTo(map);

    marker.on("click", function(){

        const nombre = lugar.tags?.name || "Lugar sin nombre";

        document.getElementById("ubicacion").textContent = nombre;

        document.getElementById("coordenadas").innerHTML =
        `🌎 Latitud: ${latitud.toFixed(6)}<br>
         🧭 Longitud: ${longitud.toFixed(6)}`;

        ubicacionActual = {
            nombre,
            lat: latitud,
            lon: longitud
        };

        actualizarClima(latitud, longitud);
    });

    marker.bindPopup(lugar.tags?.name || "Lugar sin nombre");

    marcadoresBusqueda.push(marker);
});
    alert(`Se encontraron ${data.elements.length} lugares`);
}
  
document.getElementById("btnRestaurantes").onclick = () =>
    buscarLugares("restaurant");

document.getElementById("btnGasolineras").onclick = () =>
    buscarLugares("fuel");

document.getElementById("btnHospitales").onclick = () =>
    buscarLugares("hospital");

document.getElementById("btnEscuelas").onclick = () =>
    buscarLugares("school");

document.getElementById("btnHoteles").onclick = () =>
    buscarLugares("hotel");

document.getElementById("btnTiendas").onclick = () =>
    buscarLugares("shop");

document.getElementById("btnTurismo").onclick = () =>
    buscarLugares("tourism");

  document.getElementById("btnClima").onclick = () => {
    const centro = map.getCenter();
    actualizarClima(centro.lat, centro.lng);
};
</script>
<script>

async function login() {

    const { data, error } =
        await supabaseClient.auth.signInWithOAuth({
            provider: "google",
            options: {
                redirectTo: window.location.origin
            }
        });

    if (error) {
        alert("❌ Error al iniciar sesión: " + error.message);
    }

}


async function logout() {

    const { error } =
        await supabaseClient.auth.signOut();

    if (error) {
        alert("❌ Error al cerrar sesión: " + error.message);
    }

}


supabaseClient.auth.onAuthStateChange(
    (event, session) => {

        const usuario =
            document.getElementById("usuario");

        if (session && session.user) {

            usuario.innerHTML =
                "👤 " +
                (session.user.email || "Usuario");

        } else {

            usuario.innerHTML =
                "No has iniciado sesión";

        }

    }
);

</script>
</body>
</html>

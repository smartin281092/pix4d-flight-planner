<script>
    import {onMount} from 'svelte';
    import {browser} from '$app/environment';

    let map;
    let markerLayers;
    let lineLayers;
    let markerLocations = [];
    let eye = true;
	let lines = true;
    let idStorage = 0;
    let storage = [];
    const initialView = {lat: 46.205616513508744, lng: 6.145048141479493};

    onMount(async () => {
        if (browser) {
            const L = await import('leaflet');

            for ( var i = 0, len = localStorage.length; i < len; ++i ) {
                storage = [
                    ...storage,
                    {id: i, markers: JSON.parse(localStorage[`${i+1}`])}
                ];
            }

            // center the map on Geneva
            function createMap(container) {
                let m = L.map(container, {preferCanvas: true }).setView(initialView, 18);
                L.tileLayer(
                    'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
                    {
                        attribution: `&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>`,
                        maxZoom: 25,
                    }
                ).addTo(m);

                return m;
            }

            function createLines() {
                return L.polyline(markerLocations, { color: '#E4E', opacity: 0.5 });
            }

            function createMarker(latlng) {
                L.circle(latlng, {
                    color: 'red',
                    fillColor: '#f03',
                    fillOpacity: 0.5,
                    radius: 1
                }).addTo(map);
            }

            function mapAction(container) {
                map = createMap(container); 
                
                markerLayers = L.layerGroup()
                for(let location of markerLocations) {
                    let m = createMarker(location);
                    markerLayers.addLayer(m);
                }
                
                lineLayers = createLines();
                
                markerLayers.addTo(map);
                lineLayers.addTo(map);
            }

            mapAction(document.querySelector('#map'));

            map.on('click', (e) => {
                L.circle(e.latlng, {
                    color: 'red',
                    fillColor: '#f03',
                    fillOpacity: 0.5,
                    radius: 1
                }).addTo(map);

                markerLocations.push(e.latlng);

                if(markerLocations.length > 1) {
                    L.polyline(markerLocations, { color: '#E4E', opacity: 0.5 }).addTo(map);
                }
            });

            function loadPath(id) {
                map.eachLayer((layer) => {
                    if(layer._latlng || layer._latlngs) {
                        map.removeLayer(layer);
                    }
                });

                for(let path of storage) {
                    if(path.id == id.dataset.id) {

                        markerLocations = path.markers.markers;
                        markerLayers = L.layerGroup();

                        for(let i = 0; i < markerLocations.length; i++) {
                            let m = L.circle(markerLocations[i], {
                                color: 'red',
                                fillColor: '#f03',
                                fillOpacity: 0.5,
                                radius: 1
                            }).addTo(map);

                            markerLayers.addLayer(m);
                        }
                        
                        lineLayers = L.polyline(markerLocations, { color: '#E4E', opacity: 0.5 });
                        
                        markerLayers.addTo(map);
                        lineLayers.addTo(map);
                    }
                }
            }

            setTimeout(()=>{
                let oldPaths = document.querySelectorAll('.old-path');
                for(let el of oldPaths) {
                    el.addEventListener('click', function(){
                        loadPath(el);
                    })
                }
            }, 100)

        }
    });

    $: if(markerLayers && map) {
        if(eye) {
            markerLayers.addTo(map);
        } else {
            markerLayers.remove();
        }
    }
    
    $: if(lineLayers && map) {
        if(lines) {
            lineLayers.addTo(map);
        } else {
            lineLayers.remove();
        }
    }

    function resizeMap() {
        if(map) { map.invalidateSize(); }
    }

    function savePath() {
        let idStorage = localStorage.length;
        localStorage.setItem(`${idStorage + 1}`, JSON.stringify({id: idStorage + 1, markers: markerLocations}));
        storage = [
                    ...storage,
                    {id: idStorage + 1, markers: {id: idStorage + 1, markers: markerLocations}}
                ];

        markerLocations = [];

        map.eachLayer((layer) => {
            if(layer._latlng || layer._latlngs) {
                map.removeLayer(layer);
            }
        });

        setTimeout(()=>{
            let oldPaths = document.querySelectorAll('.old-path');

            oldPaths[oldPaths.length-1].addEventListener('click', (el) => {
                map.eachLayer((layer) => {
                    if(layer._latlng || layer._latlngs) {
                        map.removeLayer(layer);
                    }
                });

                for(let path of storage) {
                    if(path.id == el.target.dataset.id) {

                        markerLocations = path.markers.markers;
                        markerLayers = L.layerGroup();

                        for(let i = 0; i < markerLocations.length; i++) {
                            let m = L.circle(markerLocations[i], {
                                color: 'red',
                                fillColor: '#f03',
                                fillOpacity: 0.5,
                                radius: 1
                            }).addTo(map);

                            markerLayers.addLayer(m);
                        }
                        
                        lineLayers = L.polyline(markerLocations, { color: '#E4E', opacity: 0.5 });
                        
                        markerLayers.addTo(map);
                        lineLayers.addTo(map);
                    }
                }
            })
        }, 100)
    }


</script>
<svelte:window on:resize={resizeMap} />

<style>
    @import url('https://fonts.googleapis.com/css2?family=Barlow:wght@200;700&display=swap');
    #map {
        width: 100vw;
        height: 100vh;
    }
    :global(html, body, #svelte) {
      width: 100%;
      height: 100%;
      margin: 0;
      padding: 0;
    }
    #menu {
        position: relative;
        z-index: 1000;
        background: rgba(0,0,0,0.3);
        backdrop-filter: blur(10px);
        padding: 20px;
        border-radius: 20px;
        border: 2px solid rgba(255, 255, 255, 0.651);
    }
    * {
        font-family: 'Barlow', sans-serif;
    }
    .old-path {
        font-weight: 200;
        color: rgba(255,255,255,0.6);
        text-align: center;
        margin: 10px 0px;
    }
    .old-path::after {
        background: rgba(255,255,255,0.2);
        width: 100%;
        height: 1px;
        display: block;
        margin-top: 10px;
        content: '';
    }
    #save {
        text-align: center;
        font-weight: 700;
        color: rgba(255,255,255,0.8);
    }
    .underline-save {
        width: calc(100% + 10px);
        height: 30px;
        border-radius: 5px;
        background: rgba(255,20,50,1);
        position: absolute;
        z-index: 999;
        margin-top: -45px;
        margin-left: -5px;
    }
    #container-menu {
        position: fixed;
        top: 80px;
        left: 13px;
        z-index: 999;
    }
</style>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css"
   integrity="sha512-xwE/Az9zrjBIphAcBb3F6JVqxf46+CDLwfLMHloNu6KEQCAWi6HcDUbeOfBIptF7tcCzusKFjFw2yuvEpDL9wQ=="
   crossorigin=""/>
<div id="map" />

<div id=container-menu>
    <div id="menu">
        {#each storage as path }
            <div class="old-path" data-id={path.id}>Load Path {path.id}</div>
        {/each}
        <div on:click={savePath} id="save">Save Path</div>
    </div>
    <div class="underline-save"></div>
    
</div>

<script>
import { onMount } from "svelte";

const distance = 300

let stations

async function getClosestStations(position){
  const response = await fetch(`https://opendata.paris.fr/api/records/1.0/search/?dataset=velib-disponibilite-en-temps-reel&q=&rows=9999&facet=name&facet=is_installed&facet=is_renting&facet=is_returning&facet=nom_arrondissement_communes&geofilter.distance=${position.latitude},${position.longitude},${distance}`)
  const data = await response.json()

  const stations = data.records.map(record => {
    console.log(record.fields)
    return {
      nom: record.fields.name,
      electrique: record.fields.ebike,
      mecanique: record.fields.mechanical,
      places: record.fields.numdocksavailable,
      distance: record.fields.dist,
      position: {
        latitude: record.fields.coordonnees_geo[0],
        longitude: record.fields.coordonnees_geo[1]
      }
    }
  })

  console.log(stations.length)

  return stations
}

function getLocation(){
  navigator.geolocation.getCurrentPosition(position => {
    stations = getClosestStations({
      latitude: position.coords.latitude,
      longitude: position.coords.longitude
    })
  })
}

onMount(() => {
  getLocation()
})
</script>

<button on:click={getLocation}>Me localiser</button>

{#if stations}
{#await stations}
Chargement
{:then data}
<ul>
{#each data as station}
  <li>
    <h2>{station.nom}</h2>
    <div class="distance">{Math.round(station.distance)}m</div>
    <div class="velib electrique">{station.electrique}</div>
    <div class="velib mecanique">{station.mecanique}</div>
    <div class="velib places">{station.places}</div>
  </li>
{/each}
</ul>
{:catch error}
Erreur {error}
{/await}
{/if}
<script context="module">
  export async function load({ params, fetch }) {
    const mapId = params.mapId || '68mcxUe8zJ2Hufws'

    const url = `https://api.allmaps.org/maps/${mapId}`
    const georef = await fetch(url).then((response) => response.json())

    const imageUri = georef.image.uri

    const imageInfo = await fetch(`${imageUri}/info.json`).then((response) => response.json())

    return {
      props: {
        georef,
        imageInfo
      }
    }
  }
</script>

<script>
  import { onMount, onDestroy } from 'svelte'

  import 'ol/ol.css'
  import Feature from 'ol/Feature.js'
  import Geolocation from 'ol/Geolocation.js'
  import Map from 'ol/Map.js'
  import Point from 'ol/geom/Point.js'
  import View from 'ol/View.js'
  import CircleStyle from 'ol/style/Circle.js'
  import Fill from 'ol/style/Fill.js'
  import Stroke from 'ol/style/Stroke.js'
  import Style from 'ol/style/Style.js'
  import VectorSource from 'ol/source/Vector.js'
  import IIIF from 'ol/source/IIIF.js'
  import IIIFInfo from 'ol/format/IIIFInfo.js'
  import TileLayer from 'ol/layer/Tile.js'
  import VectorLayer from 'ol/layer/Vector.js'

  import { createTransformer, toImage } from '@allmaps/transform'

  export let georef
  export let imageInfo

  let olContainer
  let positionFeature

  function updatePosition(coordinates) {
    if (positionFeature && coordinates && transformer) {
      const imageCoordinates = toImage(transformer, coordinates)
      positionFeature.setGeometry(new Point([imageCoordinates[0], -imageCoordinates[1]]))
    }
  }

  onMount(async () => {
    const tileLayer = new TileLayer()

    positionFeature = new Feature()

    positionFeature.setStyle(
      new Style({
        image: new CircleStyle({
          radius: 6,
          fill: new Fill({
            color: '#f309CC'
          }),
          stroke: new Stroke({
            color: '#fff',
            width: 1
          })
        })
      })
    )

    const vectorLayer = new VectorLayer({
      source: new VectorSource({
        features: [positionFeature]
      })
    })

    const map = new Map({
      layers: [tileLayer, vectorLayer],
      target: olContainer
    })

    const options = new IIIFInfo(imageInfo).getTileSourceOptions()
    options.zDirection = -1
    const iiifTileSource = new IIIF(options)
    tileLayer.setSource(iiifTileSource)
    map.setView(
      new View({
        resolutions: iiifTileSource.getTileGrid().getResolutions(),
        extent: iiifTileSource.getTileGrid().getExtent(),
        constrainOnlyCenter: true
      })
    )
    map.getView().fit(iiifTileSource.getTileGrid().getExtent())

    const geolocation = new Geolocation({
      tracking: true,
      trackingOptions: {
        enableHighAccuracy: true
      }
    })

    geolocation.on('change:position', function () {
      const coordinates = geolocation.getPosition()
      updatePosition(coordinates)
    })

    geolocation.on('error', (error) => {
      console.error(error)
    })

    geolocation.setTracking(true)
  })

  const transformer = createTransformer(georef.gcps)
</script>

<svelte:head>
  <title>Allmaps Here</title>
</svelte:head>

<div bind:this={olContainer} class="map" />

<style>
  :global(html, body, #svelte) {
    box-sizing: border-box;
    margin: 0;
    height: 100%;
  }

  :global(#svelte) {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .map {
    width: 100%;
    height: 100%;
  }
</style>

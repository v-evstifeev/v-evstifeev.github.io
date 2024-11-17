<html lang="en">
<head>
<meta charset="UTF-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Create a time slider</title>
<script src="https://cdn.maptiler.com/maptiler-sdk-js/v2.3.0/maptiler-sdk.umd.js"></script>
<link href="https://cdn.maptiler.com/maptiler-sdk-js/v2.3.0/maptiler-sdk.css" rel="stylesheet" />
<style>
  body {margin: 0; padding: 0;}
  #map {position: absolute; top: 0; bottom: 0; width: 100%;}
</style>
</head>
<body>
<style>
    .map-overlay {
        font: 12px/20px 'Helvetica Neue', Arial, Helvetica, sans-serif;
        position: absolute;
        width: 25%;
        top: 0;
        left: 0;
        padding: 10px;
    }

    .map-overlay .map-overlay-inner {
        background-color: #fff;
        box-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
        border-radius: 3px;
        padding: 10px;
        margin-bottom: 10px;
    }

    .map-overlay h2 {
        line-height: 24px;
        display: block;
        margin: 0 0 10px;
    }

    .map-overlay .legend .bar {
        height: 10px;
        width: 100%;
        background: linear-gradient(to right, #fca107, #7f3121);
    }

    .map-overlay input {
        background-color: transparent;
        display: inline-block;
        width: 100%;
        position: relative;
        margin: 0;
        cursor: ew-resize;
    }
</style>

<div id="map"></div>

<div class="map-overlay top">
    <div class="map-overlay-inner">
        <h2>Концентрация хлорофилла 2024</h2>
        <label id="month"></label>
        <input id="slider" type="range" min="0" max="11" step="1" value="0" />
    </div>
    <div class="map-overlay-inner">
        <div id="legend" class="legend">
            <div class="bar"></div>
            <div>Хлорофилл (мкг/л)</div>
        </div>
    </div>
</div>

<script src="https://d3js.org/d3.v3.min.js" charset="utf-8"></script>
<script>
    maptilersdk.config.apiKey = 'V3KVjYtkCfIVDjRVi52t';
    var map = new maptilersdk.Map({
        container: 'map',
        style: maptilersdk.MapStyle.STREETS,
        center: [38.19, 43.58],
        zoom: 6.9
    });

    var months = [
        'Январь',
        'Февраль',
        'Март',
        'Апрель',
        'Май',
        'Июнь',
        'Июль',
        'Август',
        'Сентябрь',
        'Октябрь',
        'Ноябрь',
        'Декабрь'
    ];

    function filterBy(month) {
        var filters = ['==', 'month', month];
        map.setFilter('data-circles', filters);

        // Set the label to the month
        document.getElementById('month').textContent = months[month];
    }

    map.on('load', function () {
        d3.json(
            'https://raw.githubusercontent.com/v-evstifeev/data/main/test_geo_data_m4.geojson',
            function (err, data) {
                if (err) throw err;

                // Create a month property value based on time
                // used to filter against.
                data.features = data.features.map(function (d) {
                    d.properties.month = new Date(d.properties.time*1000).getMonth();//домножаем на 1000 timestamp
					console.log(d.properties.month);
                    return d;
                });

                map.addSource('geodata', {
                    'type': 'geojson',
                    data: data
                });

                map.addLayer({
                    'id': 'data-circles',
                    'type': 'circle',
                    'source': 'geodata',
                    'paint': {
                        'circle-color': [
                            'interpolate',
                            ['linear'],
                            ['get', 'level'],
                            1,
                            '#FCA107',
                            50,
                            '#7F3121'
                        ],
                        'circle-opacity': 0.75,
                        'circle-radius': 20
                    }
                });
				
				map.on('click', 'data-circles', function (e) {
				    var text = Number.parseFloat(e.features[0].properties.level).toFixed(2);
					new maptilersdk.Popup()
						.setLngLat(e.lngLat)
						.setHTML(text.concat(' мкг/л'))
						.addTo(map);
				});

				map.on('mouseenter', 'data-circles', function () {
					map.getCanvas().style.cursor = 'pointer';
				});

				map.on('mouseleave', 'data-circles', function () {
					map.getCanvas().style.cursor = '';
				});

                filterBy(0);

                document
                    .getElementById('slider')
                    .addEventListener('input', function (e) {
                        var month = parseInt(e.target.value, 10);
						console.log(month);
                        filterBy(month);
                    });
            }
        );
    });
</script>
</body>
</html>

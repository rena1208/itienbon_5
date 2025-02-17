<script lang="ts">
	// Amplifyの設定を確かめるためのコメント
	import maplibregl, { type Map } from 'maplibre-gl';
	// import { type Point, type Position } from 'geojson';
	// import maplibregl from 'maplibre-gl';
	import OpacityControl from 'maplibre-gl-opacity';
	import 'maplibre-gl/dist/maplibre-gl.css';
	import 'maplibre-gl-opacity/dist/maplibre-gl-opacity.css';
	import { onMount } from 'svelte';

	// mapの初期設定
	const INIT_MAP_SETTING = {
		zoom: 5,
		center: [138, 37] as [number, number],
		minZoom: 5,
		maxZoom: 18,
		maxBounds: [
			[122, 20],
			[154, 50]
		] as [[number, number], [number, number]]
	} as const;

	// let mapInstance: Map = new Map(INIT_MAP_SETTING);
	let mapInstance: Map | null = null;

	let userLocation = $state(null);

	// mapの初期描画
	onMount(async () => {
		const map = new maplibregl.Map({
			container: 'map' as string,
			style: {
				version: 8,
				sources: {
					osm: {
						type: 'raster',
						tiles: ['https://tile.openstreetmap.org/{z}/{x}/{y}.png'],
						maxzoom: 19,
						tileSize: 256,
						attribution:
							'&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
					},
					// ハザードマップ
					// 洪水浸水想定区域
					hazard_flood: {
						type: 'raster',
						tiles: [
							'https://disaportaldata.gsi.go.jp/raster/01_flood_l2_shinsuishin_data/{z}/{x}/{y}.png'
						],
						minzoom: 2,
						maxzoom: 17,
						tileSize: 256,
						attribution:
							'<a href="https://disaportal.gsi.go.jp/hazardmap/copyright/opendata.html">ハザードマップポータルサイト</a>'
					},
					// 高潮浸水想定
					hazard_heightide: {
						type: 'raster',
						tiles: [
							'https://disaportaldata.gsi.go.jp/raster/03_hightide_l2_shinsuishin_data/{z}/{x}/{y}.png'
						],
						minzoom: 2,
						maxzoom: 17,
						tileSize: 256,
						attribution:
							'<a href="https://disaportal.gsi.go.jp/hazardmap/copyright/opendata.html">ハザードマップポータルサイト</a>'
					},
					// 津波浸水想定
					hazard_tsunami: {
						type: 'raster',
						tiles: [
							'https://disaportaldata.gsi.go.jp/raster/04_tsunami_newlegend_data/{z}/{x}/{y}.png'
						],
						minzoom: 2,
						maxzoom: 17,
						tileSize: 256,
						attribution:
							'<a href="https://disaportal.gsi.go.jp/hazardmap/copyright/opendata.html">ハザードマップポータルサイト</a>'
					},
					// 土砂災害警戒区域（土石流）
					hazard_doseki: {
						type: 'raster',
						tiles: [
							'https://disaportaldata.gsi.go.jp/raster/05_dosekiryukeikaikuiki/{z}/{x}/{y}.png'
						],
						minzoom: 2,
						maxzoom: 17,
						tileSize: 256,
						attribution:
							'<a href="https://disaportal.gsi.go.jp/hazardmap/copyright/opendata.html">ハザードマップポータルサイト</a>'
					},
					// 土砂災害警戒区域（急傾斜地の崩壊）
					hazard_kyukeisya: {
						type: 'raster',
						tiles: [
							'https://disaportaldata.gsi.go.jp/raster/05_kyukeishakeikaikuiki/{z}/{x}/{y}.png'
						],
						minzoom: 2,
						maxzoom: 17,
						tileSize: 256,
						attribution:
							'<a href="https://disaportal.gsi.go.jp/hazardmap/copyright/opendata.html">ハザードマップポータルサイト</a>'
					},
					// 土砂災害警戒区域（地すべり）
					hazard_jisuberi: {
						type: 'raster',
						tiles: [
							'https://disaportaldata.gsi.go.jp/raster/05_jisuberikeikaikuiki/{z}/{x}/{y}.png'
						],
						minzoom: 2,
						maxzoom: 17,
						tileSize: 256,
						attribution:
							'<a href="https://disaportal.gsi.go.jp/hazardmap/copyright/opendata.html">ハザードマップポータルサイト</a>'
					},
					skhb: {
						// 指定緊急避難場所ベクトルタイル
						type: 'vector',
						tiles: [`${location.origin}/tiles/skhb/{z}/{x}/{y}.pbf`],
						minzoom: 5,
						maxzoom: 8,
						attribution:
							'<a href="https://www.gsi.go.jp/bousaichiri/hinanbasho.html" target="_blank">国土地理院:指定緊急避難場所データ</a>'
					}
				},
				layers: [
					{
						id: 'osm-layer',
						source: 'osm',
						type: 'raster'
					},
					{
						id: 'hazard_flood-layer',
						source: 'hazard_flood',
						type: 'raster',
						paint: {
							'raster-opacity': 0.7
						},
						layout: {
							visibility: 'none'
						}
					},
					{
						id: 'hazard_hightide-layer',
						source: 'hazard_heightide',
						type: 'raster',
						paint: {
							'raster-opacity': 0.7
						},
						layout: {
							visibility: 'none'
						}
					},
					{
						id: 'hazard_tsunami-layer',
						source: 'hazard_tsunami',
						type: 'raster',
						paint: {
							'raster-opacity': 0.7
						},
						layout: {
							visibility: 'none'
						}
					},
					{
						id: 'hazard_doseki-layer',
						source: 'hazard_doseki',
						type: 'raster',
						paint: {
							'raster-opacity': 0.7
						},
						layout: {
							visibility: 'none'
						}
					},
					{
						id: 'hazard_kyukeisha-layer',
						source: 'hazard_kyukeisya',
						type: 'raster',
						paint: {
							'raster-opacity': 0.7
						},
						layout: {
							visibility: 'none'
						}
					},
					{
						id: 'hazard_jisuberi-layer',
						source: 'hazard_jisuberi',
						type: 'raster',
						paint: {
							'raster-opacity': 0.7
						},
						layout: {
							visibility: 'none'
						}
					},
					{
						id: 'skhb-1-layer',
						source: 'skhb',
						'source-layer': 'skhb',
						type: 'circle',
						paint: {
							'circle-color': '#6666cc',
							'circle-radius': [
								'interpolate',
								['linear'],
								['zoom'],
								5, // zoomレベル５の時に
								2, // 半径２px
								14, // zoomレベル１４の時に
								20 // 半径６px
							],
							'circle-stroke-width': 1,
							'circle-stroke-color': '#ffffff'
						},
						filter: ['get', '洪水'],
						layout: { visibility: 'none' }
					},
					{
						id: 'skhb-2-layer',
						source: 'skhb',
						'source-layer': 'skhb',
						type: 'circle',
						paint: {
							'circle-color': '#6666cc',
							'circle-radius': [
								'interpolate',
								['linear'],
								['zoom'],
								5, // zoomレベル５の時に
								2, // 半径２px
								14, // zoomレベル１４の時に
								20 // 半径６px
							],
							'circle-stroke-width': 1,
							'circle-stroke-color': '#ffffff'
						},
						filter: ['get', '崖崩れ、土石流及び地滑り'],
						layout: { visibility: 'none' }
					},
					{
						id: 'skhb-3-layer',
						source: 'skhb',
						'source-layer': 'skhb',
						type: 'circle',
						paint: {
							'circle-color': '#6666cc',
							'circle-radius': [
								'interpolate',
								['linear'],
								['zoom'],
								5, // zoomレベル５の時に
								2, // 半径２px
								14, // zoomレベル１４の時に
								20 // 半径６px
							],
							'circle-stroke-width': 1,
							'circle-stroke-color': '#ffffff'
						},
						filter: ['get', '高潮'],
						layout: { visibility: 'none' }
					},
					{
						id: 'skhb-4-layer',
						source: 'skhb',
						'source-layer': 'skhb',
						type: 'circle',
						paint: {
							'circle-color': '#6666cc',
							'circle-radius': [
								'interpolate',
								['linear'],
								['zoom'],
								5, // zoomレベル５の時に
								2, // 半径２px
								14, // zoomレベル１４の時に
								20 // 半径６px
							],
							'circle-stroke-width': 1,
							'circle-stroke-color': '#ffffff'
						},
						filter: ['get', '地震'],
						layout: { visibility: 'none' }
					},
					{
						id: 'skhb-5-layer',
						source: 'skhb',
						'source-layer': 'skhb',
						type: 'circle',
						paint: {
							'circle-color': '#6666cc',
							'circle-radius': [
								'interpolate',
								['linear'],
								['zoom'],
								5, // zoomレベル５の時に
								2, // 半径２px
								14, // zoomレベル１４の時に
								20 // 半径６px
							],
							'circle-stroke-width': 1,
							'circle-stroke-color': '#ffffff'
						},
						filter: ['get', '津波'],
						layout: { visibility: 'none' }
					},
					{
						id: 'skhb-6-layer',
						source: 'skhb',
						'source-layer': 'skhb',
						type: 'circle',
						paint: {
							'circle-color': '#6666cc',
							'circle-radius': [
								'interpolate',
								['linear'],
								['zoom'],
								5, // zoomレベル５の時に
								2, // 半径２px
								14, // zoomレベル１４の時に
								20 // 半径６px
							],
							'circle-stroke-width': 1,
							'circle-stroke-color': '#ffffff'
						},
						filter: ['get', '大規模な火事'],
						layout: { visibility: 'none' }
					},
					{
						id: 'skhb-7-layer',
						source: 'skhb',
						'source-layer': 'skhb',
						type: 'circle',
						paint: {
							'circle-color': '#6666cc',
							'circle-radius': [
								'interpolate',
								['linear'],
								['zoom'],
								5, // zoomレベル５の時に
								2, // 半径２px
								14, // zoomレベル１４の時に
								20 // 半径６px
							],
							'circle-stroke-width': 1,
							'circle-stroke-color': '#ffffff'
						},
						filter: ['get', '内水氾濫'],
						layout: { visibility: 'none' }
					},
					{
						id: 'skhb-8-layer',
						source: 'skhb',
						'source-layer': 'skhb',
						type: 'circle',
						paint: {
							'circle-color': '#6666cc',
							'circle-radius': [
								'interpolate',
								['linear'],
								['zoom'],
								5, // zoomレベル５の時に
								2, // 半径２px
								14, // zoomレベル１４の時に
								20 // 半径６px
							],
							'circle-stroke-width': 1,
							'circle-stroke-color': '#ffffff'
						},
						filter: ['get', '火山現象'],
						layout: { visibility: 'none' }
					}
				]
			},
			...INIT_MAP_SETTING
		});

		// Mapを更新
		mapInstance = map;

		map.on('load', () => {
			// mapの初期ロード完了時に発火するイベントの定義
			const opacity = new OpacityControl({
				baseLayers: {
					'hazard_flood-layer': '洪水浸水想定区域',
					'hazard_hightide-layer': '高潮浸水想定区域',
					'hazard_tsunami-layer': '津波浸水想定区域',
					'hazard_doseki-layer': '土石流警戒区域',
					'hazard_kyukeisha-layer': '急傾斜警戒区域',
					'hazard_jisuberi-layer': '地滑り警戒区域'
				}
			});
			map.addControl(opacity, 'top-left');

			const skhbOpacity = new OpacityControl({
				baseLayers: {
					'skhb-1-layer': '洪水',
					'skhb-2-layer': '崖崩れ、土石流及び地滑り',
					'skhb-3-layer': '高潮',
					'skhb-4-layer': '地震',
					'skhb-5-layer': '津波',
					'skhb-6-layer': '大規模な火事',
					'skhb-7-layer': '内水氾濫',
					'skhb-8-layer': '火山現象'
				}
			});
			map.addControl(skhbOpacity, 'top-right');

			// 現在位置の表示
			const geolocationControl = new maplibregl.GeolocateControl({
				trackUserLocation: true,
			});
			map.addControl(geolocationControl, 'bottom-right');
		});



		// クリックしてアラートを表示
		mapInstance.on('click', (e) => {
			const features = map.queryRenderedFeatures(e.point, {
				layers: [
					'skhb-1-layer',
					'skhb-2-layer',
					'skhb-3-layer',
					'skhb-4-layer',
					'skhb-5-layer',
					'skhb-6-layer',
					'skhb-7-layer',
					'skhb-8-layer'
				]
			});
			// alert('クリックされました');
			if (features.length === 0) return;

			// 地物があればポップアップを表示する

			const feature = features[0];
			console.log(features);
			console.log(feature);
			if (feature.geometry.type === 'Point') {
				// const popup = new maplibregl.Popup().setLngLat(feature.geometry.coordinates)

				// const coordinates = feature.geometry.coordinates as [number, number];
				// const popup = new maplibregl.Popup()
				// 	.setLngLat(coordinates)
				// 	.setHTML(

				const popup = new maplibregl.Popup()
					.setLngLat(feature.geometry.coordinates as [number, number]) // [lon, lat]
					// 名称・住所・備考・対応している災害種別を表示するよう、HTMLを文字列でセット
					.setHTML(
						`\
        <div style="font-weight:900; font-size: 1.2rem;">${feature.properties['施設・場所名']}</div>\
        <div>${feature.properties['住所']}</div>\
        <div>${feature.properties['備考'] ?? ''}</div>\
        <div>\
        <span${feature.properties['洪水'] ? '' : ' style="color:#ccc;"'}">洪水</span>\
        <span${
					feature.properties['崖崩れ、土石流及び地滑り'] ? '' : ' style="color:#ccc;"'
				}> 崖崩れ/土石流/地滑り</span>\
        <span${feature.properties['高潮'] ? '' : ' style="color:#ccc;"'}> 高潮</span>\
        <span${feature.properties['地震'] ? '' : ' style="color:#ccc;"'}> 地震</span>\
        <div>\
        <span${feature.properties['津波'] ? '' : ' style="color:#ccc;"'}>津波</span>\
        <span${feature.properties['大規模な火事'] ? '' : ' style="color:#ccc;"'}> 大規模な火事</span>\
        <span${feature.properties['内水氾濫'] ? '' : ' style="color:#ccc;"'}> 内水氾濫</span>\
        <span${feature.properties['火山現象'] ? '' : ' style="color:#ccc;"'}> 火山現象</span>\
        </div>`
					)
					.addTo(map);
			}
		});

		// 地図上でマウスが移動した際のイベント
		map.on('mousemove', (e) => {
			// 避難場所とマウスカーソルが重なった際にカーソルを変更
			const features = map.queryRenderedFeatures(e.point, {
				layers: [
					'skhb-1-layer',
					'skhb-2-layer',
					'skhb-3-layer',
					'skhb-4-layer',
					'skhb-5-layer',
					'skhb-6-layer',
					'skhb-7-layer',
					'skhb-8-layer'
				]
			});
			if (features.length > 0) {
				map.getCanvas().style.cursor = 'pointer';
			} else {
				map.getCanvas().style.cursor = '';
			}
		});
	});
</script>

<!-- id="map"がないとcontainer: 'map'が表示されない -->
<div id="map" class="app h-screen"></div>

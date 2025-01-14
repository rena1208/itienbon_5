<script lang="ts">
	import maplibregl, { type Map } from 'maplibre-gl';
	import 'maplibre-gl/dist/maplibre-gl.css';
	import { onMount } from 'svelte';

	// mapの初期設定
	const INIT_MAP_SETTING = {
		zoom: 5 as number,
		center: [138, 37] as [number, number],
		minZoom: 5 as number,
		maxZoom: 18 as number,
		maxBounds: [122, 20, 154, 50] as [number, number, number, number]
	} as const;

	let mapInstance: Map; // mapのインスタンス

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
					}
				},
				layers: [
					{
						id: 'osm-layer',
						source: 'osm',
						type: 'raster'
					}
				]
			},
			...INIT_MAP_SETTING
		});
	});

	// mapの初期化
</script>

<!-- id="map"がないとcontainer: 'map'が表示されない -->
<div id="map" class="app h-screen"></div>

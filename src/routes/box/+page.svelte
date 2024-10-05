<script lang="ts">
	let c = 95.7;
	let b = 72.7;
	let a = 54.7;
	let w = 1.8;

	let gDeg = 120;

	$: gamma = (gDeg / 180) * Math.PI;

	$: gAng = Math.PI + gamma;
	$: dward = Math.sin(gAng) * b;
	$: cosb = Math.cos(gAng) * b;
	$: xLen = a + cosb;

	$: clen = Math.pow(Math.pow(xLen, 2) + Math.pow(dward, 2), 0.5);
	$: cRate = dward / clen;

	$: beta = xLen > 0 ? -Math.asin(cRate) : Math.PI + Math.asin(cRate);
	// let beta = Math.asin(Math.cos(Math.atan((a / b - Math.cos(gamma)) / Math.sin(gamma))));
	$: alpha = Math.PI - gamma - beta;

	$: sa = Math.abs(Math.sin(beta)) * a;
	$: ca = Math.cos(beta) * a;
	$: cb = Math.abs(Math.cos(alpha)) * b;
	$: totalWidth = ca + cb;

	$: oreach = (totalWidth - c) / 2;

	$: dshiftL = Math.sin(beta - Math.PI / 2) * w;
	$: lhShift = Math.cos(beta - Math.PI / 2) * w;
	$: dshiftR = Math.sin(alpha - Math.PI / 2) * w;
	$: rhShift = Math.cos(alpha - Math.PI / 2) * w;

	$: innerReach = (innerWidth - c) / 2;
	$: extraDshiftL = innerReach > 0 ? innerReach * Math.sin(alpha) : 0;
	$: extraDshiftR = innerReach > 0 ? innerReach * Math.sin(beta) : 0;

	$: dshift = dshiftL + extraDshiftL;

	$: innerWidth = totalWidth - lhShift - rhShift;

	$: cutoff = Math.tan((Math.PI - gamma) / 2) * w;
	$: totalHeight = sa - dshift;
	$: shiftSlope = dshiftR + extraDshiftR - dshiftL - extraDshiftL;
	// $: grandRot = Math.atan(shiftSlope / innerWidth);
	$: grandRot = 0;

	const toDeg = (r: number) => (r * 180) / Math.PI;
	const cutPath = (h: number, w: number, co1: number, co2: number) =>
		`M 0 0 h ${w} l ${-co1} ${h} h -${w - co1 - co2} z`;
</script>

<container>
	<div class="controls">
		<div>
			<span>
				angle: {gDeg}
			</span>
			<input type="range" min="20" max="179" bind:value={gDeg} />
		</div>
		<div>
			<span> a: </span>
			<input type="number" min="1" max="200" step="0.05" bind:value={a} />
		</div>
		<div>
			<span> b: </span>
			<input type="number" min="1" max="200" step="0.05" bind:value={b} />
		</div>
		<div>
			<span> c: </span>
			<input type="number" min="1" max="200" step="0.05" bind:value={c} />
		</div>
		<div>
			<span> width: </span>
			<input type="number" min="0.1" max="5" step="0.05" bind:value={w} />
		</div>
	</div>

	<div class="stats">
		<span>
			tip cutoff (a & b): {cutoff.toFixed(4)}
		</span>
		<span>
			total (outer) height: {totalHeight.toFixed(3)}
		</span>
		<span>
			total (outer) width: {totalWidth.toFixed(3)}
		</span>
	</div>

	<svg viewBox="-50 -150 200 200" height="100%" width="100%">
		<rect height={w} width={c} />
		<g transform="rotate({toDeg(grandRot)}) translate({-oreach}, {dshift}) ">
			<g transform="rotate({toDeg(-beta)})">
				<path d={cutPath(w, a, cutoff, 0)} />
			</g>
			<g transform="translate({ca}, {-sa}) rotate({toDeg(alpha)})">
				<path d={cutPath(w, b, 0, cutoff)} />
			</g>
		</g>
	</svg>
</container>

<style>
	.controls {
		display: flex;
	}

	.controls > div {
		display: flex;
		flex-direction: column;
		padding: 20px;
	}

	.stats > span {
		padding: 10px;
	}

	container {
		display: flex;
		flex-direction: column;
		width: 100%;
		height: 100svh;
	}

	rect,
	path {
		fill: none;
		/* border: 1px solid black; */
		/* border-color: black; */
		/* border-width: 1px; */
		stroke: black;
		stroke-width: 0.2px;
	}
</style>

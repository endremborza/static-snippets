<script lang="ts">
	function getOreach(a: number, b: number, c: number, g: number) {
		let p1 = 2 * (1 + Math.cos(g));
		let p2 = 2 * (-b - a) * (1 + Math.cos(g));
		let p3 = -Math.pow(c, 2) + Math.pow(a, 2) + Math.pow(b, 2) + 2 * Math.cos(g) * a * b;
		return (-p2 - Math.pow(Math.pow(p2, 2) - 4 * p1 * p3, 0.5)) / (2 * p1);
	}

	let c = 95.7;
	let b = 72.7;
	let a = 54.7;
	let w = 1.8;

	let gDeg = 120;

	$: gamma = (gDeg / 180) * Math.PI;
	$: cutoff = Math.tan((Math.PI - gamma) / 2) * w;

	$: innerA = a - cutoff;
	$: innerB = b - cutoff;

	$: gAng = Math.PI + gamma;

	$: angledOreach = getOreach(innerA, innerB, c, gAng);

	$: corrOreach = Math.max(0, angledOreach);

	$: cFitA = innerA - corrOreach;
	$: cFitB = innerB - corrOreach;

	$: dward = Math.sin(gAng) * cFitB;
	$: cosb = Math.cos(gAng) * cFitB;
	$: xLen = cFitA + cosb;

	$: clen = Math.pow(Math.pow(xLen, 2) + Math.pow(dward, 2), 0.5);
	// $: cOver = clen - c;

	$: cRate = dward / clen;

	$: beta = xLen > 0 ? -Math.asin(cRate) : Math.PI + Math.asin(cRate);
	// let beta = Math.asin(Math.cos(Math.atan((a / b - Math.cos(gamma)) / Math.sin(gamma))));
	$: alpha = Math.PI - gamma - beta;

	$: sa = Math.abs(Math.sin(beta)) * a;
	$: ca = Math.cos(beta) * a;
	$: cb = Math.abs(Math.cos(alpha)) * b;
	$: totalWidth = ca + cb;

	$: betaR = beta + Math.PI / 2;
	$: dshift = -Math.sin(betaR) * w + Math.sin(beta) * corrOreach;
	$: horeach = Math.cos(betaR) * w - Math.cos(beta) * angledOreach;

	$: totalHeight = sa - dshift;

	const toDeg = (r: number) => (r * 180) / Math.PI;
	const cutPath = (h: number, w: number, co1: number, co2: number) =>
		`M 0 0 h ${w} l ${-co1} ${h} h -${w - co1 - co2} z`;
</script>

<container style="--pw: {(totalWidth / 1200).toFixed(4)}px;">
	<div class="controls">
		<div>
			<span>
				angle: {gDeg}
			</span>
			<input type="range" min="40" max="179" bind:value={gDeg} />
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
		<span>
			overhang lenght: {angledOreach.toFixed(3)}
		</span>
	</div>

	<svg viewBox="{Math.min(0, horeach) * 1.2} -{totalHeight * 1.15} {totalWidth * 1.1} {totalHeight *
			1.3}" height="100%" width="100%">
		<rect height={w} width={c} />
		<g transform="translate({horeach}, {dshift}) ">
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

	.controls>div {
		display: flex;
		flex-direction: column;
		padding: 20px;
	}

	.stats>span {
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
		fill: #825b32;
		stroke: black;
		stroke-width: var(--pw);
	}
</style>

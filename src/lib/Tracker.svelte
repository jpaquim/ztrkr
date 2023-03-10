<script lang="ts">
	import { createEventDispatcher } from 'svelte';
	import type { KeysMode, Mode, Track } from './state';
	import {
		probabilityToString,
		scaleToString,
		t16,
		type N16,
		type T16,
		type Tuple16,
	} from './utils';

	const dispatch = createEventDispatcher();

	export let mode: Mode;
	export let keysMode: KeysMode;
	export let pulseMode: boolean;
	export let helpMode = false;
	export let tracks: Tuple16<Track>;
	export let activeTracks: Set<number>;
	// export let activeTracks: Set<N16>;
	export let selectedTrack: N16;
	export let page: number;
	export let pageSize = 16;
	export let patternLength: number;
	export let patternScale: number | undefined;
	export let lengths: number[] | undefined;
	export let scales: number[] | undefined;
	export let steps: T16;
	export let fractions: T16;

	$: tracksSteps = tracks.map((track, t) =>
		Array.from({ length: lengths?.[t] ?? patternLength }, (_, s) => track.steps[s]),
	);
	$: maxSteps = t16.map(t => patternLength * (scales?.[t] ?? 1));

	$: len = helpMode ? 3 : 5;
	$: pad = (s: number | string) => String(s).padStart(len).slice(-len);

	function clickStep(step: number, track: number) {
		dispatch('step-toggle', { step, track });
	}

	function clickTrack(track: number) {
		dispatch('track-change', track);
	}
</script>

<ol class="flex" style:--page-size={pageSize} on:keydown on:keypress on:keyup>
	{#each tracks as track, t}
		<li
			class="track"
			class:selected={t === selectedTrack}
			class:active={pulseMode && tracks[t].steps[steps[t]]}
			class:inactive={!activeTracks.has(t)}
			style:--opacity={!pulseMode ? 1 : 1 - 0.5 * fractions[t]}
			on:pointerdown={() => clickTrack(t)}
			on:pointerenter={event => event.buttons !== 0 && clickTrack(t)}
		>
			<button
				><pre>{#if helpMode}c:{/if}{pad(track.channel + 1)}</pre>
				<pre>{#if helpMode}p:{/if}{pad(probabilityToString(track.probability))}</pre>
				<pre>{#if helpMode}s:{/if}{pad(scaleToString(scales?.[t] ?? patternScale ?? 1))}</pre>
				<pre>{#if helpMode}l:{/if}{pad(lengths?.[t] ?? patternLength)}</pre></button
			>
			<ol>
				{#each tracksSteps[t] as trig, s}
					<li
						on:pointerdown={() => clickStep(s, t)}
						on:pointerenter={event => event.buttons !== 0 && clickStep(s, t)}
						on:keydown
					>
						<button
							class="row"
							class:active={steps[t] === s}
							class:highlight={s % (4 * (scales?.[t] ?? patternScale ?? 1)) === 0}
							class:note={trig?.type === 'note'}
							class:lock={trig?.type === 'lock'}
							class:inactive={s >= maxSteps[t]}
							class:page-start={t === selectedTrack && s == page * pageSize}
							>{s.toString(16).padEnd(2)}{trig ? '***' : '---'}</button
						>
					</li>
				{/each}
			</ol>
		</li>
	{/each}
</ol>

<style>
	.flex {
		display: flex;
	}

	.track {
		border-color: var(--v7);
		border-width: 2px 1px;
		border-radius: 1px;
		height: 100%;
		background-color: var(--v1);
		color: var(--v9);
	}

	.track:nth-child(4n + 1) {
		color: var(--vb);
	}

	.track:nth-child(4n + 1) > button {
		box-shadow: inset 0 0 1px 1px var(--v9);
		background-color: var(--v3);
	}

	.track:first-child {
		border-left-width: 2px;
	}

	.track:last-child {
		border-right-width: 2px;
	}

	.track.selected {
		background-color: var(--v3);
		color: var(--vd);
		font-weight: bold;
	}

	.track > button {
		border-color: var(--v7);
		border-bottom-width: 1px;
		width: 100%;
		padding: 0.2em 0;
	}

	.track.selected > button {
		position: relative;
	}

	.track.selected > button::before {
		content: ' ';
		position: absolute;
		top: 0;
		bottom: 0;
		left: 0;
		right: 0;
		border-color: var(--hf);
		border-width: 2px;
		box-shadow: inset 0 0 0 1px var(--hf), 0 0 0 2px var(--hf);
		pointer-events: none;
	}

	.track.active > button {
		color: var(--vf);
	}

	.inactive {
		opacity: 20%;
	}

	.row {
		padding: 0.1em 0.3em;
		white-space: nowrap;
	}

	.row.active {
		background-color: var(--v5);
		opacity: var(--opacity);
	}

	.selected .row.active {
		background-color: var(--v7);
	}

	.highlight {
		background-color: var(--v3);
	}

	.selected .highlight {
		background-color: var(--v5);
	}

	.note {
		background-color: var(--v7);
		color: var(--v1);
	}

	.selected .note {
		background-color: var(--hf);
		color: var(--vf);
	}

	.note.active {
		background-color: var(--vd);
		color: var(--v1);
	}

	.selected .note.active {
		background-color: var(--vf);
	}

	.page-start {
		position: relative;
	}

	.page-start::before {
		content: ' ';
		position: absolute;
		height: calc(var(--page-size) * 100%);
		top: 0;
		bottom: 0;
		left: 0;
		right: 0;
		border-color: var(--hf);
		border-width: 2px;
		box-shadow: 0 0 0 2px var(--hf);
		pointer-events: none;
	}
</style>

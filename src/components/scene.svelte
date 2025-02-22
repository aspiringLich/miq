<script>
	export let scene;
	export let state;
	export let live = false;
	import { connectionMode, currentConnectionStatus, ConnectionStatusEnum, oscConfig } from "../lib/stores";
	import MeterCanvas from "./meterCanvas.svelte";
	import { createEventDispatcher } from "svelte";
	
	const dispatch = createEventDispatcher();
</script>

<div class="scene" class:live>
	<div class="mics">
		<h2 class="horiz">
			{#if live}
				<span class="liveBadge">Live</span>
			{/if}
			<span>{scene ? scene.name : "--"}</span>
		</h2>
		<div class="channels">
			{#each Object.keys(scene?.mics || {}) as i}
				<div
					class="channel"
					class:accent={scene?.mics[i]?.active}
					class:dne={!scene?.mics[i]}
					class:meteringEnabled={$connectionMode === "osc" &&
						$currentConnectionStatus.status === ConnectionStatusEnum.CONNECTED &&
						$oscConfig.liveMetersEnabled}
				>
					<!-- <div class="channelMeter" style:height={`calc(100% * ${$channelMeters[i - 1]})`} /> -->
					<MeterCanvas channel={i} />
					<div class="channel-headings">
						<h3 style="font-weight: 400; text-overflow: clip;">{i}</h3>
						<button
							class="emergency-mute"
							class:active={$state.forceMute[i]}
							class:inactive={!$state.forceMute[i]}
							on:click={() => {
								$state.forceMute[i] = !$state.forceMute[i];
								dispatch("forceMuteToggle", { channel: i });
							}}
						>
							{#if $state.forceMute[i]}
								<box-icon name="microphone-off" color="currentColor"></box-icon>
							{:else}
								<box-icon name="microphone" color="currentColor"></box-icon>
							{/if}
						</button>
					</div>
					<div>
						<p
							class="actorLabel"
							class:bigLabel={scene?.mics[i]?.character?.startsWith("#")}
							class:actorChanging={scene?.mics[i]?.switchingFrom}
							style="z-index: 10;"
						>
							<span>
								{#if scene?.mics[i]?.switchingFrom}
									{scene?.mics[i]?.switchingFrom || ""}
									<br /> <strong>&rarr; {scene?.mics[i]?.actor || ""}</strong>
								{:else}
									{scene?.mics[i]?.actor || ""}
								{/if}
							</span>
						</p>
						<p	 class={scene?.mics[i]?.character?.startsWith("#") ? "smallLabel" : "bigLabel"}>
							{scene?.mics[i]?.character || ""}
						</p>
					</div>
				</div>
			{/each}
		</div>
	</div>
	<div style="overflow: auto; position: relative;">
		<div class="notes">
			<h3 style="margin-block: 0.2em;">Notes</h3>
			<p style="overflow: auto; white-space: pre-line;">
				{scene?.notes || ""}
			</p>
		</div>
	</div>
</div>

<style lang="scss">
	.channel-headings {
		display: flex;
		justify-content: space-between;
	}

	.emergency-mute {
		padding: 0;
		margin: 0;
		border: none;
		transition: color 0.3s;

		box-icon {
			margin: auto;
		}

		&.inactive {
			color: transparent;
			&:hover {
				color: #ff00007f;
			}
		}

		&.active {
			color: #ff0000;
		}
	}

	.scene {
		display: grid;
		grid-template-columns: 4fr 1fr;
		gap: var(--spacing-required);
		padding: var(--spacing);
		border-radius: var(--rounding);
		border: 2px solid var(--green);
		opacity: var(--opacity);
		// :global(.miniMode) & {
		// 	height: unset;
		// 	max-height: 10em;
		// }
	}
	.mics {
		display: flex;
		flex-direction: column;
		gap: var(--spacing);
	}
	.scene.live {
		border-color: var(--red);
	}
	.channels {
		display: grid;
		gap: var(--spacing);
		grid-template-columns: repeat(8, 1fr);
		grid-auto-rows: 1fr;
		:global(.miniMode) & {
			grid-auto-rows: unset;
		}
	}
	.channel {
		height: 4em;
		border-radius: calc(var(--rounding) * 0.8);
		padding: calc(var(--spacing) * 0.8);
		border: 2px solid var(--fg);
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		min-width: 0;
		overflow: hidden;
		transition: 120ms;
		h3,
		p {
			white-space: nowrap;
			text-overflow: ellipsis;
			overflow: hidden;
		}
		:global(.miniMode) & {
			height: unset;
		}
		* {
			z-index: 30;
		}
		position: relative;
	}

	.actorLabel {
		border: 4px solid transparent;
		border-radius: calc(var(--rounding) * 1 / 3);
		position: relative;
		top: 4px;
		right: 4px;
		width: 100%;
		> * {
			z-index: 30;
		}
		&.actorChanging {
			transition: 360ms;
			transition-delay: 120ms;
			background: #fe0;
			color: #000;
		}
	}

	.meteringEnabled .actorLabel.actorChanging {
		background: #ff04;
		color: white;
	}

	.bigLabel {
		font-size: 0.9em;
		font-weight: 600;
	}
	.smallLabel,
	.actorLabel:not(.bigLabel) {
		font-size: 0.8em;
		font-weight: 200;
	}

	.accent {
		border-color: transparent;
	}
	.dne {
		opacity: 0.2;
	}
	.notes {
		// overflow: auto;
		position: absolute;
		top: 0;
		left: 0;
		min-width: 0;
		// max-height: 100%;
		// white-space: pre-line;
	}
	.liveBadge {
		background: var(--red);
		color: var(--red-text);
		font-weight: 600;
		text-transform: uppercase;
		font-size: 0.75em;
		padding: 0.1em 0.2em;
		border-radius: min(0.2em, var(--rounding));
		vertical-align: text-bottom;
	}
	.channelMeter {
		background: #fff4;
		position: absolute;
		width: 100%;
		bottom: 0;
		left: 0;
	}
</style>

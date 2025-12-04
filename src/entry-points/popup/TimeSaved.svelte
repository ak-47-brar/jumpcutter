<script lang="ts">
  import { tippyActionAsyncPreload as tippy } from "./tippyAction";
  import { fromS } from 'hh-mm-ss';
  import { assertNever, getMessage } from "@/helpers";
  import type { TelemetryMessage } from '@/entry-points/content/AllMediaElementsController';
  import type { Settings } from "@/settings";
  import { tweened } from "svelte/motion";
  import { linear as EasingLinear } from "svelte/easing";
  import {
    getTimeSavedComparedToIntrinsicSpeedFraction,
    getTimeSavedComparedToSoundedSpeedFraction,
  } from "@/helpers/timeSavedMath";

  type RequiredSettings = Pick<
    Settings,
    | "soundedSpeed"
    | "timeSavedRepresentation"
    | "timeSavedAveragingMethod"
    | "timeSavedAveragingWindowLength"

    | "lifetimeTimeSavedComparedToSoundedSpeed"
    | "lifetimeTimeSavedComparedToIntrinsicSpeed"
    | "lifetimeWouldHaveLastedIfSpeedWasSounded"
    | "lifetimeWouldHaveLastedIfSpeedWasIntrinsic"
  >;
  type RequiredTelemetry = Pick<
    TelemetryMessage,
    | 'elementRemainingIntrinsicDuration'
    | 'sessionTimeSaved'
    | 'lifetimeTimeSaved'
  >

  export let latestTelemetryRecord: RequiredTelemetry | undefined;
  export let settings: RequiredSettings;
  export let onSettingsChange: (newValues: Partial<RequiredSettings>) => void

  function mmSs(s: number): string {
    return fromS(Math.round(s), 'mm:ss');
  }
  
  function hhMmSs(s: number): string {
    const hours = Math.floor(s / 3600);
    const mins = Math.floor((s % 3600) / 60);
    const secs = Math.floor(s % 60);
    
    if (hours > 0) {
      return `${hours}:${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    }
    return `${mins}:${secs.toString().padStart(2, '0')}`;
  }

  $: r = latestTelemetryRecord;
  $: s = latestTelemetryRecord?.sessionTimeSaved;
  
  // Calculate total time skipped in current video (absolute value)
  $: videoTimeSkipped = s?.timeSavedComparedToSoundedSpeed ?? 0;
  $: videoTimeSkippedFormatted = hhMmSs(videoTimeSkipped);
  
  // Calculate playback rate equivalents for ETA calculation
  function getTimeSavedPlaybackRateEquivalents(
    r: RequiredTelemetry['sessionTimeSaved'] | undefined,
    settings: RequiredSettings
  ): [comparedToSounded: number, comparedToIntrinsic: number] {
    if (!r) {
      const dummyEffectiveSpeed = 1;
      return [
        dummyEffectiveSpeed,
        dummyEffectiveSpeed * settings.soundedSpeed,
      ];
    }
    const lastedActually = r.wouldHaveLastedIfSpeedWasSounded - r.timeSavedComparedToSoundedSpeed;
    if (lastedActually === 0) {
      return [1, 1];
    }
    return [
      r.wouldHaveLastedIfSpeedWasSounded / lastedActually,
      r.wouldHaveLastedIfSpeedWasIntrinsic / lastedActually,
    ]
  }
  
  $: timeSavedPlaybackRateEquivalents = getTimeSavedPlaybackRateEquivalents(s, settings);
  
  // Calculate ETA (estimated time to complete)
  $: estimatedRemainingDuration = settings &&
                                  r?.elementRemainingIntrinsicDuration != undefined &&
                                  r.elementRemainingIntrinsicDuration < Infinity
    ? (r.elementRemainingIntrinsicDuration / timeSavedPlaybackRateEquivalents[0]) / settings.soundedSpeed
    : undefined;
  
  $: estimatedRemainingFormatted = estimatedRemainingDuration != undefined
    ? hhMmSs(estimatedRemainingDuration)
    : '--:--';
  
  $: estimatedCompletionTime = estimatedRemainingDuration != undefined
    ? new Date(Date.now() + estimatedRemainingDuration * 1000).toLocaleTimeString()
    : '--:--';

  // Lifetime time saved (tweened for smooth animation)
  let currLifetimeSavedVal: number | undefined =
    r?.lifetimeTimeSaved.timeSavedComparedToSoundedSpeed
  type NewVal = number;
  const tweenedLifetimeTimeSavedComparedToSoundedSpeed =
    tweened<number | undefined>(currLifetimeSavedVal, {
      easing: EasingLinear,
      duration: (from, to) => {
        const diffMs = (
          (to as NewVal) -
          (from as NewVal)
        ) * 1000;
        const relativeRate = 1;
        return Math.min(
          diffMs / relativeRate,
          5_000,
        )
      },
    })
  $: if (r) {
    const newVal: NewVal = r.lifetimeTimeSaved.timeSavedComparedToSoundedSpeed
    const prevVal = currLifetimeSavedVal
    currLifetimeSavedVal = newVal
    if (newVal !== prevVal) {
      tweenedLifetimeTimeSavedComparedToSoundedSpeed.set(newVal)
    }
  }
  $: isTweenedLifetimeTimeSavedIncreasing =
    r != undefined
    && $tweenedLifetimeTimeSavedComparedToSoundedSpeed != undefined
    && $tweenedLifetimeTimeSavedComparedToSoundedSpeed <
      r.lifetimeTimeSaved.timeSavedComparedToSoundedSpeed

  const commonTippyProps = {
    theme: "my-tippy white-space-pre-line",
    placement: "bottom",
    hideOnClick: false,
  } as const
</script>

<div class="time-display">
  <!-- Video Time Skipped -->
  <div class="metric-row">
    <button
      type="button"
      use:tippy={{
        ...commonTippyProps,
        content: "Total time skipped in this video by fast-forwarding through silent parts"
      }}
      class="metric-label"
    >
      <span>⏭️ Skipped:</span>
    </button>
    <span class="metric-value primary">{videoTimeSkippedFormatted}</span>
  </div>

  <!-- ETA to Complete -->
  {#if estimatedRemainingDuration != undefined && estimatedRemainingDuration < 10000 * 60 * 60}
  <div class="metric-row">
    <button
      type="button"
      use:tippy={{
        ...commonTippyProps,
        content: `Estimated time remaining to finish this video\nWill complete at: ${estimatedCompletionTime}`
      }}
      class="metric-label"
    >
      <span>⏱️ ETA:</span>
    </button>
    <span class="metric-value primary">{estimatedRemainingFormatted}</span>
  </div>
  {/if}

  <!-- Lifetime Total (smaller, at bottom) -->
  <div class="metric-row lifetime">
    <button
      type="button"
      use:tippy={{
        ...commonTippyProps,
        content: 'Total time saved since installation'
      }}
      class="metric-label"
    >
      <span>💾 Total:</span>
    </button>
    <span
      class="metric-value lifetime-time-saved"
      class:green={isTweenedLifetimeTimeSavedIncreasing}
    >{fromS(
      $tweenedLifetimeTimeSavedComparedToSoundedSpeed
        ?? settings.lifetimeTimeSavedComparedToSoundedSpeed,
      'mm:ss.sss'
    ).slice(0, -1)}</span>
  </div>
</div>

<style>
  .time-display {
    display: flex;
    flex-direction: column;
    gap: 8px;
    padding: 4px 0;
  }

  .metric-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
  }

  .metric-row.lifetime {
    margin-top: 4px;
    padding-top: 8px;
    border-top: 1px solid rgba(128, 128, 128, 0.3);
    font-size: 0.9em;
    opacity: 0.85;
  }

  button {
    border: none;
    padding: 0;
    background: unset;
    font: inherit;
    cursor: help;
  }

  .metric-label {
    text-align: left;
    white-space: nowrap;
  }

  .metric-value {
    font-weight: 600;
    font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
    white-space: nowrap;
  }

  .metric-value.primary {
    font-size: 1.1em;
    color: #4CAF50;
  }

  @media (prefers-color-scheme: light) {
    .metric-value.primary {
      color: #2E7D32;
    }
    .metric-row.lifetime {
      border-top-color: rgba(0, 0, 0, 0.2);
    }
  }

  .lifetime-time-saved {
    transition-property: color, text-shadow;
    transition-timing-function: ease-out;
    transition-duration: 500ms, 15s;
  }
  
  .lifetime-time-saved.green {
    color: #b0ffb0;
    text-shadow:
      0px 0px 8px #b0ffb0,
      0px 0px 8px #b0ffb0;
    transition-duration: 10ms, 10s;
  }
  
  @media (prefers-color-scheme: light) {
    .lifetime-time-saved.green {
      color: #008000;
      text-shadow: 0px 0px 8px #00FF0050;
    }
  }
</style>
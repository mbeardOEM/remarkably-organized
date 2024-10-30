<script lang="ts">
	import { formatToString, getFirstDayOfWeek, type Timeframe } from '$lib';

	let { timeframe = {} as Timeframe, startWeekOnSunday = false } = $props();

	const weekStart = $derived(
		new Date(getFirstDayOfWeek(timeframe.start, startWeekOnSunday)),
	);

	const startHour = 6;
	const endHour = 18; // 6 PM
	const intervals = Array.from({ length: (endHour - startHour) * 4 }, (_, i) => ({
		hour: startHour + Math.floor(i / 4),
		minute: (i % 4) * 15,
	}));

	// Define working hours with precise 15-minute rounding logic
	const workingHours = {
		monToThu: { start: 7, end: 16.5 }, // 7 AM to 4:30 PM
		friday: { start: 7, end: 11 }, // 7 AM to 11 AM
	};
</script>

<div class="week">
	<!-- Empty space to align hour labels and day headers -->
	<div class="empty-space"></div>

	<!-- Day headers -->
	{#each new Array(7) as _, dayIndex}
		{@const date = new Date(weekStart.getTime() + dayIndex * 86400000)}
		<div class="day-header">
			{date.toLocaleString('default', { weekday: 'short', timeZone: 'UTC' })}
			{@html formatToString(date.getUTCDate(), { type: 'ordinal', html: true })}
		</div>
	{/each}

	<!-- Hour labels -->
	{#each Array.from({ length: endHour - startHour }) as _, i}
		<div class="hour-label">
			{(i + startHour) % 12 === 0 ? 12 : (i + startHour) % 12}
			<small>{i + startHour < 12 ? 'AM' : 'PM'}</small>
		</div>
	{/each}

	<!-- 15-minute intervals for each day of the week -->
	{#each new Array(7) as _, dayIndex}
		{@const date = new Date(weekStart.getTime() + dayIndex * 86400000)}
		{#each intervals as interval, intervalIndex}
			<!-- Corrected working hour calculation -->
			{@const isWorkingHour =
				(dayIndex >= 0 &&
					dayIndex <= 3 && // Monday to Thursday
					interval.hour + interval.minute / 60 >= workingHours.monToThu.start &&
					interval.hour + interval.minute / 60 < workingHours.monToThu.end) ||
				(dayIndex === 4 && // Friday
					interval.hour + interval.minute / 60 >= workingHours.friday.start &&
					interval.hour + interval.minute / 60 < workingHours.friday.end)}

			<div
				class="hour-block {isWorkingHour ? '' : 'outside-working-hours'}"
				class:active={timeframe.month === date.getUTCMonth() + 1 &&
					timeframe.daySinceMonth === date.getUTCDate()}
				style="grid-column: {dayIndex + 2}; grid-row: {intervalIndex + 2};">
			</div>
		{/each}
	{/each}
</div>

<style lang="scss">
	.week {
		display: grid;
		grid-template-columns: 3rem repeat(7, 1fr); /* First column for hour labels */
		grid-template-rows: 2rem repeat(52, 1fr); /* 52 rows for 15-minute blocks from 6 AM to 6 PM */
		width: 100%;
		height: 100%;
		justify-items: stretch;
		align-items: stretch;
		grid-auto-flow: column;
		padding: 0 1rem 0 0;
	}

	/* Empty space above hour labels for alignment */
	.empty-space {
		grid-row: 1;
		grid-column: 1;
		border-bottom: solid 1px var(--outline);
	}

	/* Day header styling */
	.day-header {
		grid-row: 1; /* Keep header in the first row */
		text-align: center;
		font-weight: bold;
		padding: 0.5rem 0;
		background-color: #f0f0f0;
		border-bottom: solid 1px var(--outline);
		font-size: 0.9em;
	}

	/* Hour label styling */
	.hour-label {
		text-align: center;
		grid-column: 1;
		font-weight: var(--font-weight-light);
		font-size: 0.8em;
		color: var(--text-low);
		grid-row: span 4; /* Each hour label spans 4 rows to match 15-minute blocks */
		padding-top: 0.3rem;
		border-top: solid 1px var(--outline);
		border-bottom: solid 1px var(--outline);
		border-right: solid 1px var(--outline);

		small {
			font-size: 0.6em;
		}
	}

	/* Styling for each 15-minute block */
	.hour-block {
		border: solid 1px var(--outline);
		background-color: #ffffff;
	}

	/* Grey out blocks outside working hours */
	.outside-working-hours {
		background-color: var(--panel-low-2); /* Light grey for non-working hours */
	}

	/* Right border for the last day of the week */
	.day-header:nth-of-type(7) ~ .hour-block {
		border-right: solid 1px var(--outline);
	}
</style>

<script lang="ts">
	import { Popover } from 'bits-ui';
	import {
		CalendarDate,
		createCalendar,
		toCalendar,
		getWeeksInMonth,
		startOfMonth,
		startOfWeek,
		isEqualDay
	} from '@internationalized/date';
	import { onMount } from 'svelte';

	let { date = $bindable(), name = null, id = null } = $props();

	const now = new Date();
	const gregorianToday = new CalendarDate(now.getFullYear(), now.getMonth() + 1, now.getDate());
	const persianCalendar = createCalendar('persian');
	let current = $state(toCalendar(gregorianToday, persianCalendar));
	let selected = $state<CalendarDate | null>(null);

	const locale = 'fa-IR';
	const firstDayToken = 'sat';

	function buildMonthGrid(date: CalendarDate) {
		const firstOfMonth = startOfMonth(date);
		const weeksCount = getWeeksInMonth(date, 'fa-IR');
		const weekStart = startOfWeek(firstOfMonth, 'fa-IR', firstDayToken as any);
		const weeks: (CalendarDate | null)[][] = [];
		let cursor: CalendarDate = weekStart as CalendarDate;

		for (let w = 0; w < weeksCount; w++) {
			const week: (CalendarDate | null)[] = [];
			for (let d = 0; d < 7; d++) {
				const inMonth = cursor.year === date.year && cursor.month === date.month;
				week.push(inMonth ? cursor : null);
				cursor = cursor.add({ days: 1 });
			}
			weeks.push(week);
		}
		return weeks;
	}

	let grid = $derived(buildMonthGrid(current));

	function monthName(date: CalendarDate) {
		try {
			const dtf = new Intl.DateTimeFormat(locale, { month: 'long', year: 'numeric' });
			const greg = toCalendar(date, createCalendar('gregory'));
			const jsDate = new Date(greg.year, greg.month - 1, greg.day);
			return dtf.format(jsDate);
		} catch {
			const fallback = [
				'فروردین',
				'اردیبهشت',
				'خرداد',
				'تیر',
				'مرداد',
				'شهریور',
				'مهر',
				'آبان',
				'آذر',
				'دی',
				'بهمن',
				'اسفند'
			];
			return `${fallback[date.month - 1]} ${date.year}`;
		}
	}

	function prevMonth() {
		current = current.add({ months: -1 });
	}
	function nextMonth() {
		current = current.add({ months: 1 });
	}

	function chooseDay(d: CalendarDate) {
		selected = d;
		date = d;
	}

	const weekdayHeaders = ['شن', 'یک', 'دو', 'سه', 'چهار', 'پنج', 'جم'];
	let isOpen = $state(false);

	let inputItemRef = $state<HTMLInputElement>();
	onMount(() => {
		if (id) {
			inputItemRef!.id = id;
		}
		if (name) {
			inputItemRef!.name = name;
		}
	});
</script>

<div class="main">
	<Popover.Root bind:open={isOpen}>
		<Popover.Trigger style="background: white; padding: 0; border:0">
			<input
				bind:this={inputItemRef}
				{id}
				{name}
				type="text"
				placeholder="YYYY/MM/DD"
				value={selected !== null ? `${selected?.year}/${selected?.month}/${selected?.day}` : ''}
				class="date-input"
			/>
		</Popover.Trigger>
		<Popover.Portal>
			<Popover.Content
				onOpenAutoFocus={(e) => e.preventDefault()}
				sideOffset={8}
				class="popover-content"
			>
				<div class="calendar">
					<div class="calendar-header">
						<button onclick={prevMonth} class="nav-btn">&lt;</button>
						<div class="month-name">{monthName(current)}</div>
						<button onclick={nextMonth} class="nav-btn">&gt;</button>
					</div>

					<div class="weekdays">
						{#each weekdayHeaders as hd}
							<div>{hd}</div>
						{/each}
					</div>

					<div class="days">
						{#each grid as week}
							{#each week as day}
								{#if day}
									<button
										onclick={() => {
											chooseDay(day);
											isOpen = false;
										}}
										class="
											day-btn
											{day === selected ? ' selected' : ''}
											{day.year !== current.year || day.month !== current.month ? ' faded' : ''}
											{isEqualDay(day, toCalendar(gregorianToday, persianCalendar)) ? ' today' : ''}
										"
									>
										{day.day}
									</button>
								{:else}
									<div class="empty"></div>
								{/if}
							{/each}
						{/each}
					</div>

					<div class="footer-buttons">
						<button
							class="today-btn"
							onclick={() => {
								chooseDay(toCalendar(gregorianToday, persianCalendar));
								isOpen = false;
							}}
						>
							انتخاب امروز
						</button>
						<button
							class="clear-btn"
							onclick={() => {
								selected = null;
								date = null;
								isOpen = false;
							}}
						>
							پاک کردن
						</button>
					</div>
				</div>
			</Popover.Content>
		</Popover.Portal>
	</Popover.Root>
</div>

<style>
	:root {
		--radius: 0.625rem;
		--background: oklch(1 0 0);
		--foreground: oklch(0.129 0.042 264.695);
		--card: oklch(1 0 0);
		--card-foreground: oklch(0.129 0.042 264.695);
		--popover: oklch(1 0 0);
		--popover-foreground: oklch(0.129 0.042 264.695);
		--primary: oklch(0.208 0.042 265.755);
		--primary-foreground: oklch(0.984 0.003 247.858);
		--secondary: oklch(0.968 0.007 247.896);
		--secondary-foreground: oklch(0.208 0.042 265.755);
		--muted: oklch(0.968 0.007 247.896);
		--muted-foreground: oklch(0.554 0.046 257.417);
		--accent: oklch(0.968 0.007 247.896);
		--accent-foreground: oklch(0.208 0.042 265.755);
		--destructive: oklch(0.577 0.245 27.325);
		--border: oklch(0.929 0.013 255.508);
		--input: oklch(0.929 0.013 255.508);
		--ring: oklch(0.704 0.04 256.788);
		--chart-1: oklch(0.646 0.222 41.116);
		--chart-2: oklch(0.6 0.118 184.704);
		--chart-3: oklch(0.398 0.07 227.392);
		--chart-4: oklch(0.828 0.189 84.429);
		--chart-5: oklch(0.769 0.188 70.08);
		--sidebar: oklch(0.984 0.003 247.858);
		--sidebar-foreground: oklch(0.129 0.042 264.695);
		--sidebar-primary: oklch(0.208 0.042 265.755);
		--sidebar-primary-foreground: oklch(0.984 0.003 247.858);
		--sidebar-accent: oklch(0.968 0.007 247.896);
		--sidebar-accent-foreground: oklch(0.208 0.042 265.755);
		--sidebar-border: oklch(0.929 0.013 255.508);
		--sidebar-ring: oklch(0.704 0.04 256.788);
	}

	.dark {
		--background: oklch(0.129 0.042 264.695);
		--foreground: oklch(0.984 0.003 247.858);
		--card: oklch(0.208 0.042 265.755);
		--card-foreground: oklch(0.984 0.003 247.858);
		--popover: oklch(0.208 0.042 265.755);
		--popover-foreground: oklch(0.984 0.003 247.858);
		--primary: oklch(0.929 0.013 255.508);
		--primary-foreground: oklch(0.208 0.042 265.755);
		--secondary: oklch(0.279 0.041 260.031);
		--secondary-foreground: oklch(0.984 0.003 247.858);
		--muted: oklch(0.279 0.041 260.031);
		--muted-foreground: oklch(0.704 0.04 256.788);
		--accent: oklch(0.279 0.041 260.031);
		--accent-foreground: oklch(0.984 0.003 247.858);
		--destructive: oklch(0.704 0.191 22.216);
		--border: oklch(1 0 0 / 10%);
		--input: oklch(1 0 0 / 15%);
		--ring: oklch(0.551 0.027 264.364);
		--chart-1: oklch(0.488 0.243 264.376);
		--chart-2: oklch(0.696 0.17 162.48);
		--chart-3: oklch(0.769 0.188 70.08);
		--chart-4: oklch(0.627 0.265 303.9);
		--chart-5: oklch(0.645 0.246 16.439);
		--sidebar: oklch(0.208 0.042 265.755);
		--sidebar-foreground: oklch(0.984 0.003 247.858);
		--sidebar-primary: oklch(0.488 0.243 264.376);
		--sidebar-primary-foreground: oklch(0.984 0.003 247.858);
		--sidebar-accent: oklch(0.279 0.041 260.031);
		--sidebar-accent-foreground: oklch(0.984 0.003 247.858);
		--sidebar-border: oklch(1 0 0 / 10%);
		--sidebar-ring: oklch(0.551 0.027 264.364);
	}

	@theme inline {
		--radius-sm: calc(var(--radius) - 4px);
		--radius-md: calc(var(--radius) - 2px);
		--radius-lg: var(--radius);
		--radius-xl: calc(var(--radius) + 4px);
		--color-background: var(--background);
		--color-foreground: var(--foreground);
		--color-card: var(--card);
		--color-card-foreground: var(--card-foreground);
		--color-popover: var(--popover);
		--color-popover-foreground: var(--popover-foreground);
		--color-primary: var(--primary);
		--color-primary-foreground: var(--primary-foreground);
		--color-secondary: var(--secondary);
		--color-secondary-foreground: var(--secondary-foreground);
		--color-muted: var(--muted);
		--color-muted-foreground: var(--muted-foreground);
		--color-accent: var(--accent);
		--color-accent-foreground: var(--accent-foreground);
		--color-destructive: var(--destructive);
		--color-border: var(--border);
		--color-input: var(--input);
		--color-ring: var(--ring);
		--color-chart-1: var(--chart-1);
		--color-chart-2: var(--chart-2);
		--color-chart-3: var(--chart-3);
		--color-chart-4: var(--chart-4);
		--color-chart-5: var(--chart-5);
		--color-sidebar: var(--sidebar);
		--color-sidebar-foreground: var(--sidebar-foreground);
		--color-sidebar-primary: var(--sidebar-primary);
		--color-sidebar-primary-foreground: var(--sidebar-primary-foreground);
		--color-sidebar-accent: var(--sidebar-accent);
		--color-sidebar-accent-foreground: var(--sidebar-accent-foreground);
		--color-sidebar-border: var(--sidebar-border);
		--color-sidebar-ring: var(--sidebar-ring);
	}

	.trigger {
		background-color: red;
		/* display: inline-flex;
		align-items: center;
		justify-content: center;
		height: 40px;
		background-color: #222;
		color: #fff;
		border-radius: 6px;
		padding: 4px 8px;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.15);
		cursor: pointer;
		transition: background-color 0.2s ease; */
	}
	.trigger:hover {
		background-color: #333;
	}

	.popover-content {
		margin-top: 8px;
	}

	.calendar {
		background-color: #fff;
		border-radius: 12px;
		padding: 16px;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
		max-width: 300px;
		margin: auto;
	}

	.calendar-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 12px;
	}

	.nav-btn {
		border: none;
		background: none;
		font-weight: bold;
		color: #2563eb;
		padding: 4px 8px;
		border-radius: 4px;
		cursor: pointer;
	}
	.nav-btn:hover {
		background-color: #f0f0f0;
	}

	.month-name {
		font-size: 1.1rem;
		font-weight: 600;
	}

	.weekdays {
		display: grid;
		grid-template-columns: repeat(7, 1fr);
		text-align: center;
		color: #2563eb;
		font-size: 0.9rem;
		margin-bottom: 4px;
		direction: rtl;
	}

	.days {
		display: grid;
		grid-template-columns: repeat(7, 1fr);
		gap: 4px;
		text-align: center;
		direction: rtl;
	}

	.day-btn {
		border: none;
		border-radius: 6px;
		padding: 6px;
		background: transparent;
		cursor: pointer;
		color: #333;
		transition:
			background-color 0.2s ease,
			color 0.2s ease;
	}
	.day-btn:hover {
		background-color: #e5e5e5;
	}

	.day-btn.selected {
		background-color: #2563eb;
		color: white;
	}

	.day-btn.faded {
		opacity: 0.4;
	}

	.day-btn.today {
		background-color: #f3f4f6;
	}

	.empty {
		padding: 6px;
	}

	.footer-buttons {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 8px;
		margin-top: 10px;
	}

	.today-btn,
	.clear-btn {
		border: none;
		border-radius: 6px;
		padding: 6px;
		color: white;
		cursor: pointer;
		transition: background-color 0.2s ease;
	}

	.today-btn {
		background-color: #2563eb;
	}
	.today-btn:hover {
		background-color: #1d4ed8;
	}

	.clear-btn {
		background-color: #dc2626;
	}
	.clear-btn:hover {
		background-color: #b91c1c;
	}
	/* 
	.date-input {
		padding-top: 0.25rem;
		padding-bottom: 0.25rem;
		padding-left: 0.75rem;
		padding-right: 0.75rem;
		border-radius: 0.375rem;
		border-width: 1px;
		outline-style: none;
		width: 100%;
		min-width: 0;
		font-size: 1rem;
		line-height: 1.5rem;
		box-shadow: 0 0 0 1px rgba(0, 0, 0, 0.05);
	} */

	.date-input {
		display: flex;
		text-align: center;
		height: 2.25rem; /* h-9 */
		width: 100%; /* w-full */
		min-width: 0; /* min-w-0 */
		border-radius: 0.375rem; /* rounded-md */
		border: 1px solid var(--input); /* border border-input */
		background: var(--background); /* bg-background */
		padding-left: 0.75rem; /* px-3 */
		padding-right: 0.75rem;
		padding-top: 0.25rem; /* py-1 */
		padding-bottom: 0.25rem;
		font-size: 1rem; /* text-base */
		box-shadow: 0 1px 2px var(--shadow-xs); /* shadow-xs */
		outline: none; /* outline-none */
		transition:
			color 150ms,
			box-shadow 150ms; /* transition-[color,box-shadow] */
	}

	/* Text Selection */
	.date-input::selection {
		background: var(--primary); /* selection:bg-primary */
		color: var(--primary-foreground); /* selection:text-primary-foreground */
	}

	/* Placeholder */
	.date-input::placeholder {
		color: var(--muted-foreground); /* placeholder:text-muted-foreground */
	}

	/* Focus-visible */
	.date-input:focus-visible {
		border-color: var(--ring); /* focus-visible:border-ring */
		box-shadow: 0 0 0 3px var(--ring / 0.5); /* focus-visible:ring-[3px] + ring-ring/50 */
	}

	/* Disabled */
	.date-input:disabled {
		cursor: not-allowed; /* disabled:cursor-not-allowed */
		opacity: 0.5; /* disabled:opacity-50 */
	}

	/* ARIA invalid */
	.date-input[aria-invalid='true'] {
		border-color: var(--destructive); /* aria-invalid:border-destructive */
		box-shadow: 0 0 0 3px var(--destructive / 0.2); /* aria-invalid:ring-destructive/20 */
	}

	/* md:text-sm */
	@media (min-width: 768px) {
		.date-input {
			font-size: 0.875rem; /* text-sm */
		}
	}

	/* dark:bg-input/30 */
	@media (prefers-color-scheme: dark) {
		.date-input {
			background: rgba(var(--input-rgb), 0.3);
		}

		.date-input[aria-invalid='true'] {
			box-shadow: 0 0 0 3px var(--destructive / 0.4);
		}
	}
</style>

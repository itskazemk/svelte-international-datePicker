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

	let { date = $bindable() } = $props();

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
</script>

<div class="main">
	<Popover.Root bind:open={isOpen}>
		<Popover.Trigger style="background: white; padding: 0; border:0">
			<input
				type="text"
				class="date-input"
				placeholder="YYYY/MM/DD"
				value={selected !== null ? `${selected?.year}/${selected?.month}/${selected?.day}` : ''}
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
	.main {
		text-align: center;
		padding: 20px;
		font-family: sans-serif;
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

	.date-input {
		border: 1px solid #5b21b6;
		border-radius: 4px;
		padding: 4px 6px;
		text-align: center;
		background: #fff;
		color: #111;
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
</style>

<script lang="ts">
	import { Popover, Separator, Toggle } from 'bits-ui';

	let width = $state(1024);
	let height = $state(768);

	import { CalendarDate, createCalendar, toCalendar } from '@internationalized/date';

	import {
		// queries / helpers
		startOfMonth,
		getWeeksInMonth,
		getDayOfWeek,
		startOfWeek,
		// equality helpers
		isEqualDay
	} from '@internationalized/date';

	// ---------- state (Svelte 5 runes) ----------
	// current Gregorian date -> convert to Persian calendar using createCalendar('persian')
	const now = new Date();
	const gregorianToday = new CalendarDate(now.getFullYear(), now.getMonth() + 1, now.getDate());

	// create a Persian Calendar instance (use the built-in identifier 'persian')
	const persianCalendar = createCalendar('persian');

	// convert the Gregorian CalendarDate to the Persian calendar
	let current = $state(toCalendar(gregorianToday, persianCalendar));
	let selected = $state<CalendarDate | null>(null);

	// $inspect(selected?.toString());

	// config: locale string used by Intl (for month/weekday formatting if needed)
	const locale = 'fa-IR-u-ca-persian';
	// first day of week as a DayOfWeek index where 0 is the locale's first day.
	// We'll compute offsets via getDayOfWeek which returns 0..6 where 0 === first weekday for the locale.
	// For Iran/Sat-first, we'll use 'sat' when calling startOfWeek/getDayOfWeek if needed.
	const firstDayToken = 'sat'; // 'sun'|'mon'|'tue'|'wed'|'thu'|'fri'|'sat'

	// ---------- helpers (use correct API) ----------
	// buildMonthGrid: returns array of weeks, each week an array of DateValue | null
	function buildMonthGrid(date: CalendarDate) {
		// startOfMonth(date) returns the first date of the month (in the same calendar system).
		const firstOfMonth = startOfMonth(date);

		// getWeeksInMonth(date, <locale>): number of rows the calendar will show for that month.
		// `getWeeksInMonth` uses locale to decide first-day-of-week and week counting rules.
		const weeksCount = getWeeksInMonth(date, 'fa-IR');

		// determine the starting cell: back up from firstOfMonth to the start of that week
		const weekStart = startOfWeek(firstOfMonth, 'fa-IR', firstDayToken as any);

		// iterate days filling weeksCount × 7 cells
		const weeks: (CalendarDate | null)[][] = [];
		let cursor: CalendarDate = weekStart as CalendarDate;

		for (let w = 0; w < weeksCount; w++) {
			const week: (CalendarDate | null)[] = [];
			for (let d = 0; d < 7; d++) {
				// include the day if it's in the same Persian month/year, otherwise a blank cell
				const inMonth = cursor.year === date.year && cursor.month === date.month;
				week.push(inMonth ? cursor : null);

				// advance one day by constructing a new CalendarDate using calendar arithmetic:
				// CalendarDate.add exists as an instance method (`cursor = cursor.add({ days: 1 })`)
				cursor = cursor.add({ days: 1 });
			}

			weeks.push(week);
		}

		return weeks;
	}

	// derived grid using Svelte 5 runes
	let grid = $derived(buildMonthGrid(current));

	// format month name using Intl (fallback to hard-coded Persian names if Intl doesn't support)
	function monthName(date: CalendarDate) {
		try {
			// use Intl.DateTimeFormat with Persian calendar — this relies on the environment's ICU data.
			const dtf = new Intl.DateTimeFormat(locale, { month: 'long', year: 'numeric' });
			// We need a JS Date to feed Intl. Convert the Persian CalendarDate back to Gregorian for formatting:
			// Use toCalendar in reverse by converting the Persian date to a Gregorian CalendarDate.
			// Simplest approach: create a JS Date from the equivalent Gregorian by converting to the gregory calendar:
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

	// navigation (CalendarDate is immutable; use .add())
	function prevMonth() {
		current = current.add({ months: -1 });
	}
	function nextMonth() {
		current = current.add({ months: 1 });
	}

	function chooseDay(d: CalendarDate) {
		console.log(d);
		selected = d;
	}

	// weekday headers (Intl fallback)
	const weekdayHeaders = (() => {
		try {
			const dtf = new Intl.DateTimeFormat(locale, { weekday: 'short' });
			// pick 7 consecutive JS dates (arbitrary) and format them
			const arr: string[] = [];
			for (let i = 0; i < 7; i++) arr.push(dtf.format(new Date(2020, 0, 5 + i - 1)));
			return arr;
		} catch {
			return ['ش', 'ی', 'د', 'س', 'چ', 'پ', 'ج'];
		}
	})();
</script>

<Popover.Root>
	<Popover.Trigger
		class="rounded-input bg-dark
	text-background shadow-mini hover:bg-dark/95 inline-flex h-10 items-center justify-center px-[21px] text-[15px] font-medium whitespace-nowrap transition-all select-none hover:cursor-pointer active:scale-[0.98]"
	>
		<input type="text" />
	</Popover.Trigger>
	<Popover.Portal>
		<Popover.Content
			trapFocus={false}
			onOpenAutoFocus={(e) => {
				// prevent auto focus
				e.preventDefault();
			}}
			class="border-dark-10 bg-background shadow-popover data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[side=bottom]:slide-in-from-top-2 data-[side=left]:slide-in-from-right-2 data-[side=right]:slide-in-from-left-2 data-[side=top]:slide-in-from-bottom-2 z-30 w-full max-w-[328px] origin-(--bits-popover-content-transform-origin) rounded-[12px] border p-4"
			sideOffset={8}
		>
			<div class="mx-auto max-w-md rounded-2xl bg-white p-4 shadow">
				<div class="mb-3 flex items-center justify-between">
					<button onclick={prevMonth} class="rounded px-3 py-1 hover:bg-gray-100">‹</button>
					<div class="text-center">
						<div class="text-lg font-semibold">{monthName(current)}</div>
					</div>
					<button onclick={nextMonth} class="rounded px-3 py-1 hover:bg-gray-100">›</button>
				</div>

				<!-- weekdays -->
				<div class="mb-1 grid grid-cols-7 text-center text-sm">
					{#each weekdayHeaders as hd}
						<div class="py-1">{hd}</div>
					{/each}
				</div>

				<!-- days -->
				<div class="grid grid-cols-7 gap-1 text-center">
					{#each grid as week}
						{#each week as day}
							{#if day}
								<button
									onclick={() => chooseDay(day)}
									class="rounded p-2 hover:bg-gray-100
                   {day === selected ? 'bg-blue-500 text-white' : 'bg-white text-gray-800'}
                   {day.year !== current.year || day.month !== current.month ? 'opacity-40' : ''}"
								>
									{day.day}
								</button>
							{:else}
								<div class="p-2"></div>
							{/if}
						{/each}
					{/each}
				</div>

				{#if selected}
					<div class="mt-3 text-sm text-gray-600">
						انتخاب‌شده: {selected.year}/{selected.month}/{selected.day}
					</div>
				{/if}
			</div>
		</Popover.Content>
	</Popover.Portal>
</Popover.Root>

<style>
	:global(body) {
		direction: rtl;
	}
</style>

<script>
    import Icon from "$lib/Icon.svelte";
    import { createEventDispatcher } from "svelte";
    import { scale } from "svelte/transition";
    import { clickOutside } from './utils.js';

    //0 => Single Date
    //1 => Multiple Dates
    //2 => Range of Dates (In dev)
    export let selectionType = 0;

    //0 => Weeks Starting from Sunday
    //1 => Weeks Starting from Monday
    export let weekDaysType = 0;

    //"bottom" => Calendar under button
    //"top" => Calendar above button (In dev)
    export let position = 'bottom';

    // "YY" => yearShort)
    // "DDDD" => day)
    // "DDD" => dayShort)
    // "DD" => date.toString().padStart(2, '0'))
    // "D" => date)
    // "MMMM" => month)
    // "MMM" => monthShort)
    // "MM" => monthNumber.toString().padStart(2, '0'))
    // "M" => monthNumber)
    export let dateFormat = 'DD/MM/YYYY';

    // Sample Date object => new Date('August 17, 2025 03:24:00')
    export let maxDate = null;
    export let minDate = null;

    const dispatch = createEventDispatcher();

    /* ---------- */
    let datepickSettings = {
        selectedDate: new Date(),
        selectedDates: [new Date()],
        initialSelected: new Date(),
        today: new Date(),
        minDate,
        maxDate,
        dateFormat,
        position,
        visible: false,
        weekDaysType,
        selectionType
    };

    function getWeekNumber(date) {
        const firstDayOfTheYear = new Date(date.getFullYear, 0 , 1);
        const pastDaysOfYear = (date - firstDayOfTheYear) / 86400000;
        return Math.ceil((pastDaysOfYear + firstDayOfTheYear.getDay() + 1)/7)
    }
    
    class Day { 
        constructor(date = null, lang = 'default') {
            date = date ?? new Date();
            this.Date = date;
            this.date = date.getDate();
            this.day = date.toLocaleString(lang, { weekday: 'long' });
            this.dayNumber = date.getDay() + 1;
            this.dayShort = date.toLocaleString(lang, { weekday: 'short' });
            this.year = date.getFullYear();
            this.yearShort = Number(
                date.toLocaleString(lang, { year: '2-digit' })
            );
            this.month = date.toLocaleString(lang, { month: 'long'});
            this.monthShort = date.toLocaleString(lang, { month: 'short'});
            this.monthNumber = Number(
                date.toLocaleString(lang, { month: '2-digit' })
            );
            this.week = getWeekNumber(date);
        }

        get isToday() {
            return this.isEqualTo(new Date())
        }

        isEqualTo(date) {
            date = date instanceof Day ? date.Date : date;
            return date.getDate() === this.date &&
                date.getMonth() === this.monthNumber - 1 &&
                date.getFullYear() === this.year;
        }

        format(formatStr) {
            return formatStr
                .replace(/\bYYYY\b/, this.year)
                .replace(/\bYY\b/, this.yearShort)
                .replace(/\bDDDD\b/, this.day)
                .replace(/\bDDD\b/, this.dayShort)
                .replace(/\bDD\b/, this.date.toString().padStart(2, '0'))
                .replace(/\bD\b/, this.date)
                .replace(/\bMMMM\b/, this.month)
                .replace(/\bMMM\b/, this.monthShort)
                .replace(/\bMM\b/, this.monthNumber.toString().padStart(2, '0'))
                .replace(/\bM\b/, this.monthNumber)
        }
    }

    /* ---------- */
    function isLeapYear(year) {
        return year % 100 === 0 ? year % 400 === 0 : year % 4 === 0;
    }

    class Month { 
        constructor(date = null, lang = 'default') {
            const day = new Day(date, lang);
            const monthSize = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
            this.lang = lang;
            this.name = day.month;
            this.number = day.monthNumber;
            this.year = day.year;
            this.numberOfDays = monthSize[this.number - 1];
            if (this.number === 2) {
                this.numberOfDays += isLeapYear(day.year) ? 1 : 0;
            }
            
            this[Symbol.iterator] = function* () {
                let number = 1;
                yield this.getDay(number);
                while(number < this.numberOfDays) {
                    ++number;
                    yield this.getDay(number);
                }
            }
        }
        getDay(date) {
            return new Day(new Date(this.year, this.number - 1, date), this.lang);
        }
    }

    /* ---------- */
    class Calendar {
        constructor(year = null, monthNumber = null, lang = 'default') {
            this.today = new Day(null, lang);
            this.year = year ?? this.today.year;
            this.month = new Month(new Date(this.year, (monthNumber || this.today.monthNumber) - 1), lang);
            this.lang = lang;
            this.weekDays = Array.from({length: 7});
        
            this[Symbol.iterator] = function* () {
                let number = 1;
                yield this.getMonth(number);
                while(number < 12) {
                    ++number;
                    yield this.getMonth(number);
                }
            }
            this.weekDays.forEach((_, i) => {
                const day = this.month.getDay(i + 1);
                if(!this.weekDays.includes(day.day)) {
                    if (datepickSettings.weekDaysType == 0) this.weekDays[day.dayNumber - 1] = day.day;
                    else {
                        if (day.day == "Sunday") this.weekDays[6] = day.day;
                        else this.weekDays[day.dayNumber - 2] = day.day;
                    }
                }
            })
        }

        get isLeapYear() {
            return isLeapYear(this.year);
        }

        getMonth(monthNumber) {
            return new Month(new Date(this.year, monthNumber - 1), this.lang);
        }

        getPreviousMonth() {
            if (this.month.number === 1) {
                return new Month(new Date(this.year - 1, 11), this.lang);
            }
            return new Month(new Date(this.year, (this.month.number - 1) - 1), this.lang);
        }

        getNextMonth() {
            if (this.month.number === 12) {
                return new Month(new Date(this.year + 1, 0), this.lang);
            }
            return new Month(new Date(this.year, (this.month.number + 1) - 1), this.lang);
        }

        goToNextYear() {
            this.year += 1;
            this.month = new Month(new Date(this.year, 0), this.lang);
        }

        goToPreviousYear() {
            this.year -= 1;
            this.month = new Month(new Date(this.year, 11), this.lang);
        }

        goToDate(monthNumber, year) {
            this.month = new Month(new Date(year, monthNumber - 1), this.lang);
            this.year = year;
        }

        goToNextMonth() {
            if(this.month.number === 12) {
                return this.goToNextYear();
            }
            
            this.month = new Month(new Date(this.year, (this.month.number + 1) - 1), this.lang);
        }
        
        goToPreviousMonth() {
            if(this.month.number === 1) {
                return this.goToPreviousYear();
            }
            
            this.month = new Month(new Date(this.year, (this.month.number - 1) - 1), this.lang);
        }
    }
    
    let selectedDays = [];
    $: if (datepickSettings.selectedDates) {
        updateSelectedDaysArray();
    } else { selectedDays = []; }

    function updateSelectedDaysArray() {
        let selectedDays = [];
        datepickSettings.selectedDates.forEach(date => {
            selectedDays.push(new Day(date));
        });
    }

    $: selectedDay = new Day(datepickSettings.selectedDate);
    $: selectedMonth = new Month(selectedDay.Date);
    $: calendar = new Calendar(selectedDay.year, selectedDay.monthNumber);
    
    $: monthText = selectedMonth.name;

    function toggleCalendar () {
        datepickSettings.visible = !datepickSettings.visible; 
    }

    function confirmCalendar() {
        datepickSettings.initialSelected = selectedDay.Date;
        dispatch('confirm', getAnswer());
        toggleCalendar();
    }

    function getAnswer() {
        switch (selectionType) {
            case 0:
                return { selectedDate: datepickSettings.selectedDate }
            case 1:
                return { selectedDates: datepickSettings.selectedDates }
            default:
                return;
        }
    }

    function resetCalendar () {
        if (datepickSettings.selectionType == 0) {
            datepickSettings.selectedDate = datepickSettings.initialSelected;
        } else if (datepickSettings.selectionType == 1) {
            datepickSettings.selectedDates = [new Date()];
        }
        update++;
        toggleCalendar();
    }

    function nextMonth() {
        selectedMonth = calendar.getNextMonth();
        update++;
        calendar.goToNextMonth();
    }

    function prevMonth() {
        selectedMonth = calendar.getPreviousMonth();
        update++;
        calendar.goToPreviousMonth();
    }

    let update = 0;
    function nextYear() {
        calendar.goToNextYear();
        update++;
    }

    function prevYear() {
        calendar.goToPreviousYear();
        update++;
    }

    function getMonthDaysGrid() {
        let firstDayOfTheMonth;
        if (datepickSettings.weekDaysType == 0) {
            firstDayOfTheMonth = calendar.month.getDay(1);
        } else {
            firstDayOfTheMonth = calendar.month.getDay(0);
        }
        const totalLastMonthFinalDays = firstDayOfTheMonth.dayNumber - 1;
        const totalDays = calendar.month.numberOfDays + totalLastMonthFinalDays;
        const monthList = Array.from({length: totalLastMonthFinalDays});
        for(let i = totalLastMonthFinalDays;i<totalDays;i++) {
            monthList[i] =calendar.month.getDay(i + 1 - totalLastMonthFinalDays);
        }
        return monthList;
    }

    function isSelectedDate(date, selected) {
        if (date) {
            return date.date == selected.date &&
            date.monthNumber == selected.monthNumber &&
            date.year == selected.year;
        } else return false;
    }

    function isGreaterThan(date, smallerDate) {
        if (date && smallerDate) { 
            if (date.year > smallerDate.year) {
                return true;
            } else if (date.year == smallerDate.year) {
                if (date.monthNumber > smallerDate.monthNumber) {
                    return true;
                } else if (date.monthNumber == smallerDate.monthNumber) {
                    if (date.date > smallerDate.date) {
                        return true;
                    }
                }
            }
        } else return true;
        return false;
    }

    function dayClickHandler(monthday) {
        if (datepickSettings.selectionType == 0) {
            datepickSettings.selectedDate = monthday.Date;
        } else if (datepickSettings.selectionType == 1) {
            const isSelected = (element) => {
                return element.getDate() === monthday.Date.getDate() &&
                element.getMonth() === monthday.Date.getMonth() &&
                element.getFullYear() === monthday.Date.getFullYear()
            }
            const index = datepickSettings.selectedDates.findIndex(isSelected);
            if (index === -1) {
                datepickSettings.selectedDates.push(monthday.Date);
            } else {
                datepickSettings.selectedDates.splice(index, 1);
            }
            datepickSettings.selectedDate = monthday.Date;
            
            datepickSettings.selectedDates = datepickSettings.selectedDates;
        } else {
            console.log("ERR");
        }
        update++;
    }

    function isInSelectedArray (monthday) {
        const isSelected = (element) => {
            return element.getDate() === monthday.Date.getDate() &&
            element.getMonth() === monthday.Date.getMonth() &&
            element.getFullYear() === monthday.Date.getFullYear()
        }
        if (datepickSettings.selectedDates.findIndex(isSelected) === -1) {
            return false;
        } else {
            return true;
        }
        
    }
</script>

<div class="everything" use:clickOutside on:click_outside={() => {
    if (!datepickSettings.visible) {
        return;
    }
    datepickSettings.visible = false;
}}>
    <div on:click={() => {
        toggleCalendar()
    }} class="cursor-pointer toggle_cal flex justify-center items-center">
        {#if (datepickSettings.selectionType == 0)}
            <p class="mr-15px">{selectedDay.format(datepickSettings.dateFormat)}</p>
            <Icon icon="calendar.svg"/>
        {:else if (datepickSettings.selectionType == 1)}
            {#key selectedDays}
                <p class="mr-15px">Select Dates</p>
                <Icon icon="calendar.svg"/>
            {/key}
        {/if}
    </div>
    <!--drop-down--use:clickOutside on:click_outside={handleClickOutside}-->
    {#if datepickSettings.visible}
        <div class="rounded-lg drop-down bg-blue-500 {datepickSettings.position}" transition:scale>
            <div class="inner-drop-down">
                <div class="year flex justify-center items-center">
                    <Icon icon="left_arrow.svg" className="p-0 border-8 border-transparent border-solid w-0 h-0 rounded-sm cursor-pointer relative" on:click={prevYear}/>
                    {#key update }
                        <p class="text-center w-20">
                            {calendar.year}
                        </p>
                    {/key}
                    <Icon icon="right_arrow.svg" className="next-month p-0 border-8 border-transparent border-solid w-0 h-0 rounded-sm cursor-pointer relative" on:click={nextYear}/>
                </div>
                <div class="month flex justify-center items-center mb-3">
                    <Icon icon="left_arrow.svg" className="p-0 border-8 border-transparent border-solid w-0 h-0 rounded-sm cursor-pointer relative" on:click={prevMonth}/>
                    <p class="text-center w-40">
                        {monthText}
                    </p>
                    <Icon icon="right_arrow.svg" className="next-month p-0 border-8 border-transparent border-solid w-0 h-0 rounded-sm cursor-pointer relative mb-10" on:click={nextMonth}/>
                </div>
                {#key update}
                    <div class="main-calendar">
                        <div class="week-days title">
                            {#each calendar.weekDays as weekDay, i}
                                <span>{weekDay.substring(0, 2)}</span>
                            {/each}
                        </div>
                        <div class="week-days days text-white">
                            {#each getMonthDaysGrid() as monthday}
                                {#if datepickSettings.minDate || datepickSettings.maxDate}
                                    {#if !datepickSettings.minDate}
                                        {#if monthday && isGreaterThan(new Day(datepickSettings.maxDate), monthday)}
                                            {#if datepickSettings.selectionType == 0}
                                                <div class="hover-day month-day flex justify-center items-center cursor-pointer" class:selected={isSelectedDate(monthday, selectedDay)} on:click={() => dayClickHandler(monthday)}>
                                                    {#if monthday}
                                                        <p class:today_day={isSelectedDate(monthday, new Day(datepickSettings.today))}>{monthday.date}</p>
                                                    {/if}
                                                </div>
                                            {:else if datepickSettings.selectionType == 1}
                                                <div class="hover-day month-day flex justify-center items-center cursor-pointer" class:selected={isInSelectedArray(monthday)} on:click={() => dayClickHandler(monthday)}>
                                                    {#if monthday}
                                                        <p class:today_day={isSelectedDate(monthday, new Day(datepickSettings.today))}>{monthday.date}</p>
                                                    {/if}
                                                </div>
                                            {/if}
                                        {:else}
                                            <div class="month-day flex justify-center items-center">
                                                {#if monthday}
                                                    <p class="inactive" class:today_day={isSelectedDate(monthday, new Day(datepickSettings.today))}>{monthday.date}</p>
                                                {/if}
                                            </div>
                                        {/if}
                                    {:else if !datepickSettings.maxDate}
                                        {#if monthday && isGreaterThan(monthday, new Day(datepickSettings.minDate))}
                                            {#if datepickSettings.selectionType == 0}
                                                <div class="hover-day month-day flex justify-center items-center cursor-pointer" class:selected={isSelectedDate(monthday, selectedDay)} on:click={() => dayClickHandler(monthday)}>
                                                    {#if monthday}
                                                        <p class:today_day={isSelectedDate(monthday, new Day(datepickSettings.today))}>{monthday.date}</p>
                                                    {/if}
                                                </div>
                                            {:else if datepickSettings.selectionType == 1}
                                                <div class="hover-day month-day flex justify-center items-center cursor-pointer" class:selected={isInSelectedArray(monthday)} on:click={() => dayClickHandler(monthday)}>
                                                    {#if monthday}
                                                        <p class:today_day={isSelectedDate(monthday, new Day(datepickSettings.today))}>{monthday.date}</p>
                                                    {/if}
                                                </div>
                                            {/if}
                                        {:else}
                                            <div class="month-day flex justify-center items-center">
                                                {#if monthday}
                                                    <p class="inactive" class:today_day={isSelectedDate(monthday, new Day(datepickSettings.today))}>{monthday.date}</p>
                                                {/if}
                                            </div>
                                        {/if}
                                    {:else}
                                        {#if monthday && isGreaterThan(new Day(datepickSettings.maxDate), monthday) && isGreaterThan(monthday, new Day(datepickSettings.minDate))}
                                            {#if datepickSettings.selectionType == 0}
                                                <div class="hover-day month-day flex justify-center items-center cursor-pointer" class:selected={isSelectedDate(monthday, selectedDay)} on:click={() => dayClickHandler(monthday)}>
                                                    {#if monthday}
                                                        <p class:today_day={isSelectedDate(monthday, new Day(datepickSettings.today))}>{monthday.date}</p>
                                                    {/if}
                                                </div>
                                            {:else if datepickSettings.selectionType == 1}
                                                <div class="hover-day month-day flex justify-center items-center cursor-pointer" class:selected={isInSelectedArray(monthday)} on:click={() => dayClickHandler(monthday)}>
                                                    {#if monthday}
                                                        <p class:today_day={isSelectedDate(monthday, new Day(datepickSettings.today))}>{monthday.date}</p>
                                                    {/if}
                                                </div>
                                            {/if}
                                        {:else}
                                            <div class="month-day flex justify-center items-center">
                                                {#if monthday}
                                                    <p class="inactive" class:today_day={isSelectedDate(monthday, new Day(datepickSettings.today))}>{monthday.date}</p>
                                                {/if}
                                            </div>
                                        {/if}
                                    {/if}
                                {:else}
                                    <div class="hover-day month-day flex justify-center items-center cursor-pointer" class:selected={isSelectedDate(monthday, selectedDay)} on:click={() => dayClickHandler(monthday)}>
                                        {#if monthday}
                                            <p class:today_day={isSelectedDate(monthday, new Day(datepickSettings.today))}>{monthday.date}</p>
                                        {/if}
                                    </div>
                                {/if}
                            {/each}
                        </div>
                    </div>
                {/key}
                <div class="actions flex mt-2 items-center justify-center">
                    <div class="sub-title btn cursor-pointer" on:click={() => resetCalendar()}>
                        <p>Cancel Selection</p>
                    </div>
                    <div class="title btn cursor-pointer" on:click={() => confirmCalendar()}>
                        <p>Confirm Selection</p>
                    </div>
                </div>
            </div>
        </div>
    {/if}
</div>
<style>
    .drop-down {
        position: relative;
    }

    .inner-drop-down {
        position: absolute;
        left: 0;
        width: 300px;
        box-shadow: 0 0 8px rgba(0,0,0,0.2);
        background-color: #2F3136;
        border: 2.5px solid #F52434;
        border-radius: 8px;
        padding:22px;
        margin-top: 5px;
    }

    .week-days {
        display: grid;
        grid-template-columns: repeat(7, 1fr);
        grid-gap: 8px;
    }

    .week-days span {
        display: flex;
        justify-content: center;
        align-items: center;
        text-transform: capitalize;
        font-size: 15px;
        font-style: bold;
        line-height: 22px;
        font-style: normal;
        font-weight: 500;
    }

    .month-day {
        width: 27px;
        height: 27px;
    }

    .hover-day:hover {
        background-color: grey;
        border-radius: 50%;
    }

    .days {
        margin-bottom: 0;
    }

    .selected {
        background-color: #F52434;
        border-radius: 50%;
    }

    p {
        font-size: 15px;
        font-style: bold;
        line-height: 22px;
        font-style: normal;
        font-weight: 500;
    }

    .today_day {
        border: solid 1px white;
        border-radius: 50%;
        width: 27px;
        height: 27px;
        text-align: center;
    }

    .title {
        color: #FF9742;
    }

    .btn {
        width: 100px;
        height: 34px;
        text-align: center;
        display: flex;
        align-items: center;
        padding: 5%;
        margin: 6px;
    }

    .toggle_cal {
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        padding: 4px;
        width: 300px;
        height: 32px;
        background: #F52434;
        border-radius: 8px;
    }

    .everything {
        width: 300px;
    }

    .inactive {
        color: #F52434;
    }

    .sub-title {
        color: grey;
    }
</style>
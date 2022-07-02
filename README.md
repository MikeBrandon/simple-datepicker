# Simple Datepicker
This was the first ever ambitious svelte component i worked on. I recently just moved it to sveltekit and added a few optimizations and improvements but still plan on improving it even more just for fun so welcome to its official repo.

## Technologies Used
- Sveltekit
- windicss

## Usage
Well I haven't added the package to npm yet and don't really think I plan to in the near future but you could download the `src/lib/Datepicker.svelte` file and add it to your components folder then just import it as you would other components as shown in the example below.
```html
    <Datepicker 
        selectionType={0}
        weekDaysType={0}
        position={'bottom'}
        dateFormat={"DD/MM/YYYY"}
        maxDate={new Date('August 17, 2025 03:24:00')}
        minDate={null}
        on:confirm={(e) => {
            console.log(e.detail);
        }}
    />
```
You can listen to the on:confirm comand to check for what the user has selected and use it to update you're variable. Below are example outputs of the console.log(e.detail) with different selectionTypes.

```javascript
    {
        selectedDate: Wed Jul 13 2022 00:00:00 GMT+0300 (East Africa Time)
    }
```
selectionType 1: Single Date DatePicker

```javascript
    {
        selectedDates: [
            Wed Jul 13 2022 00:00:00 GMT+0300 (East Africa Time),
            Thur Jul 14 2022 00:00:00 GMT+0300 (East Africa Time)
        ]
    }
```
selectionType 2: Multiple Date DatePicker

### Props
I have provided a couple of props to customize the datepicker. Some of which are still in development.
```javascript
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
```

## Future Plans
I plan to add a few more features to this project when I get free time and these features include:
- Range selectionType
- Click outside calendar to close calendar
- Custom Colors so users can use their own
- Top calender position relative to datepicker button
- Responsiveness fixes
- Further optimizations

Features I do not plan to add but maybe in the future when I can code like the movies.
- Time Picker

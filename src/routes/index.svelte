<script>
    import Datepicker from "$lib/Datepicker.svelte";
</script>

<div class="flex items-center justify-center h-screen bg-black">
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
</div>

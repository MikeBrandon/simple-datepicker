<script>
    export var icon;
    export var alt = "icon";
    export var className = "";
</script>

<img src="{icon}" class={`h-full flex-grow-0 flex-shrink-0 select-none${className}`} {alt} draggable={false} on:click>
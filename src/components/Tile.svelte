<script lang="ts">
    import { createEventDispatcher } from "svelte";

    export let value: string;
    export let sweeped: boolean;

    export let x: number, y: number;

    let exploded = false;
    let flag = false;

    const dispatch = createEventDispatcher();

    const colors = {
        '0': 'black',
        '1': 'blue',
        '2': `lime`,
        '3': 'red',
        '4': 'purple',
        '5': 'brown',
        '6': 'cyan',
        '7': 'white',
        '8': 'grey',
        '💣': 'black'
    }

    const click = () => {
        if(!sweeped && !flag){
            if(value === '💣') exploded = true;
            dispatch('sweep', {
                x, y 
            });
        }
    }

    const rightClick = (e: MouseEvent) => {
        e.preventDefault();
        flag = !flag;
    };
</script>

<style>
    p{
        /* height: 100%; */
        margin: 0;
        display: grid;
        place-items: center;
        font-size: 2em;     
        color: var(--color);
        user-select: none;
    }

    .hidden{
        border: 1px solid #333;
    }

    .exposed{
        background-color: #777;
    }

    .exploded{
        background-color: red;
    }
</style>

<p class={`${sweeped ? 'exposed' : 'hidden'} ${exploded ? 'exploded' : ''}` } 
    style="--color: {colors[value]}"
    on:click={click}
    on:contextmenu={rightClick}>
    {flag ? '🚩' : sweeped ? (value !== '0' ? value : '⠀') : '⠀'}
</p>
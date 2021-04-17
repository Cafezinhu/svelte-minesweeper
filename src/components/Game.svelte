<script lang='ts'>
    import type ITile from "../interfaces/Tile";

    import Tile from "./Tile.svelte";


    let columns = 12;
    let rows = 12;
    let mines = 15;

    const createGame = (columns: number, rows: number, mineCount: number):ITile[][] => {
        let newGame: ITile[][] = [];

        let tiles: {x: number, y: number}[] = [];

        for (let index = 0; index < columns; index++) {
            let row: ITile[] = [];
            for (let rowIndex = 0; rowIndex < rows; rowIndex++) {
                row.push({value: '0', sweeped: false});
                tiles.push({x: index, y: rowIndex});
            }
            newGame.push(row);
        }

        for (let index = 0; index < mineCount; index++) {
            const tile = tiles[Math.floor(Math.random() * tiles.length)];
            
            newGame[tile.x][tile.y].value = '💣';
            
            tiles = tiles.filter(iteratedTile => {
                return iteratedTile !== tile;
            });
        }

        newGame = newGame.map((row, x) => {
            return row.map((tile, y) => {
                if(tile.value !== '💣'){
                    let bombs = 0;
                    for (let ix = x - 1; ix <= x + 1; ix++) {
                        if(ix < 0 || ix >= columns) continue;
                        for (let iy = y - 1; iy <= y + 1; iy++) {
                            if(iy < 0 || iy >= rows) continue;
                            if(newGame[ix][iy].value === '💣') bombs++;
                        }
                    }
                    return {...tile, value: bombs.toString()};
                }
                return tile;
            });
        });

        return [...newGame];
    }

    let game = createGame(columns, rows, mines);

    const sweep = (e:CustomEvent<{x: number, y: number}>) => {
        const {x, y} = e.detail;
        expose(x, y);
    }

    const expose = (x: number, y: number) => {
        game[x][y].sweeped = true;
        
        let value = game[x][y].value
        if(value === '0') exposeAdjascents(x, y);
        else if(value === '💣') exposeMines();
    }

    const exposeMines = () => {
        game = game.map(row => {
            return row.map(tile => {
                return {...tile, sweeped: tile.value === '💣' ? true : tile.sweeped}
            });
        });
    }

    const exposeAdjascents = (x: number, y: number) => {
        for (let ix = x - 1; ix <= x + 1; ix++) {
            if(ix < 0 || ix >= columns) continue;
            for (let iy = y - 1; iy <= y + 1; iy++) {
                if(iy < 0 || iy >= rows) continue;
                let tile = game[ix][iy];
                if(!tile.sweeped){
                    expose(ix, iy);
                }
            }
        }
    }
</script>



<style>
    #minefield{
        height: 35em;
        width: 35em;
        border: .3em solid white;
        border-radius: .2em;
        display: grid;
        grid-template-columns: repeat(var(--columns), 1fr);
    }
</style>

<div id="minefield" style="--columns: {columns}">
    {#each game as row, x}
        {#each row as tile, y}
            <Tile value={tile.value} sweeped={tile.sweeped} x={x} y={y} on:sweep={sweep}/>
        {/each}
    {/each}
</div>
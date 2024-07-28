<script lang="ts">
    import { onDestroy, onMount } from "svelte";
    // const activeLetter = document.getElementById("activeLetter");
    
    // activeLetter?.style.left
    let x = 0;
    let y = 0;
    const step = 60;
    const letterSize = 60;
    const gameWidth = 540;
    const gameHeight = 600;
    const rows = (gameHeight / letterSize);
    const cols = (gameWidth / letterSize);
    const gameWidthString = `max-w-[${gameWidth}px]`;
    const gameHeightString = `h-[${gameHeight}px]`;
    let activeLetter = 0;
    let activeLetterText = '';
    let lastMove = 0;
    let gameSpeed = 10;
    let gameInterval: number;
    let gameGrid: string[][] = [];
    let columnDepths: number[] = [];
    let currentColumn = 0;
    let gameOn = false;


    function buildColumnDepths() {
        columnDepths = [];
        for (let i = 0; i < cols; i++) {
            columnDepths.push(rows);
        }
    }

    function buildGameGrid() {
        gameGrid = [];
        for (let i = 0; i < rows; i++) {
            let currentRow = [];
            for (let j = 0; j < cols; j++) {
                currentRow.push('.')
            }
            gameGrid.push(currentRow);
        }
    }


    const alphabet: string[] = [
        'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M',
        'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'
    ];
    let letterLocs:number[][] = [];
    let letters: HTMLDivElement[] = [];

// Function to handle div creation
    const createLetter = () => {
        letterLocs.push([4 * letterSize, 0]);
        currentColumn = 4;
        activeLetter = letterLocs.length - 1;
        const gameBox = document.getElementById('gameBox');
        const divElement = document.createElement('div');
        activeLetterText = alphabet[Math.floor(Math.random() * alphabet.length)]
        divElement.textContent = activeLetterText;
        divElement.style.position = 'absolute';
        divElement.style.backgroundColor = '#fef9c3';
        divElement.style.width = `${letterSize}px`;
        divElement.style.height = `${letterSize}px`;
        divElement.style.top = `${letterLocs[activeLetter][1]}px`;
        divElement.style.left = `${letterLocs[activeLetter][0]}px`;
        divElement.style.border = 'black 1px solid';
        divElement.style.textAlign = 'center';

        letters.push(divElement);
        if (gameBox) gameBox.appendChild(divElement);
    };

    const handleKeyDown = (event: KeyboardEvent) => {
        if (gameOn) {

            switch (event.key) {
                case "ArrowUp":
                    break;
                case "ArrowDown":
                    if(letterLocs[activeLetter][1]< (columnDepths[currentColumn] -1) * letterSize) {
                        letterLocs[activeLetter][1] = (columnDepths[currentColumn] -1) * letterSize;
                        letters[activeLetter].style.top = `${letterLocs[activeLetter][1]}px`;
                        // columnDepths[currentColumn] -= 1;
                    }
                    break;
                case "ArrowLeft":
                    if(letterLocs[activeLetter][0] > 0) {
                        letterLocs[activeLetter][0] -= step;
                        letters[activeLetter].style.left = `${letterLocs[activeLetter][0]}px`;
                        currentColumn -= 1;
                    }
                    break;
                case "ArrowRight":
                    if (letterLocs[activeLetter][0]< gameWidth - letterSize) {
                        letterLocs[activeLetter][0] += step;
                        letters[activeLetter].style.left = `${letterLocs[activeLetter][0]}px`;
                        currentColumn += 1;
                    }
                    break;
            }
        }
    };

    onMount(() => {
        window.addEventListener("keydown", handleKeyDown);
        
        return () => {
            window.removeEventListener("keydown", handleKeyDown);
        };
    });

    onDestroy(() => {
        clearInterval(gameInterval);
    })

    function startGame() {
        gameOn = true;
        const gameBox = document.getElementById('gameBox');
        if (gameBox) gameBox.innerHTML = '';
        buildGameGrid();
        buildColumnDepths();
        console.log(columnDepths);
        console.log(gameGrid);
        createLetter();
        resetGame();
    }

    function checkColumnDepths() {
        for (let depth of columnDepths) {
            if (depth < 1) {
                return false;
            }
        }
        return true;
    }

    function resetGame() {
        clearInterval(gameInterval);
        gameInterval = setInterval(() => {
            if (!checkColumnDepths()) {
                gameOn = false;
                clearInterval(gameInterval);
            }
            if (Date.now() > lastMove + gameSpeed) {
                if (letterLocs[activeLetter][1] < (columnDepths[currentColumn] -1) * letterSize) {
                    letterLocs[activeLetter][1] += 1;
                    letters[activeLetter].style.top = `${letterLocs[activeLetter][1]}px`;
                    lastMove = Date.now();
                } else {
                    gameGrid[columnDepths[currentColumn] - 1][currentColumn] = activeLetterText;
                    const foundNewWords = findAllNewWords(columnDepths[currentColumn] - 1, currentColumn);
                    columnDepths[currentColumn] -= 1;
                    console.log("words: ", foundNewWords);
                    createLetter();
                    lastMove = Date.now();
                }
            }
        }, 10)
    }

    const wordList = [
    'act', 'add', 'age', 'aim', 'air', 'all', 'and', 'any', 'ape', 'arc', 'are', 
    'arm', 'art', 'ask', 'ate', 'bag', 'bar', 'bat', 'bed', 'beg', 'bet', 'bid', 
    'big', 'bin', 'bit', 'bog', 'bow', 'box', 'boy', 'bud', 'bug', 'bun', 'bus', 
    'but', 'buy', 'cab', 'cap', 'car', 'cat', 'cow', 'cry', 'cub', 'cup', 'cut', 
    'day', 'dew', 'dig', 'dim', 'dip', 'dog', 'dot', 'dry', 'due', 'dye', 'ear', 
    'eat', 'eel', 'egg', 'end', 'far', 'fat', 'fax', 'fig', 'fin', 'fit', 'fix', 
    'fly', 'fog', 'for', 'fun', 'fur', 'gap', 'gas', 'get', 'gig', 'god', 'got', 
    'gum', 'gun', 'gut', 'gym', 'hat', 'hay', 'hem', 'her', 'him', 'hip', 'hit', 
    'hog', 'hop', 'hot', 'hug', 'hut', 'ice', 'icy', 'ink', 'inn', 'ion', 'its', 
    'jam', 'jar', 'jet', 'jig', 'job', 'jog', 'joy', 'key', 'kid', 'kit', 'lab', 
    'lap', 'law', 'lay', 'leg', 'let', 'lip', 'log', 'lot', 'low', 'mad', 'man', 
    'map', 'mat', 'mix', 'mob', 'mop', 'mud', 'mug', 'nap', 'net', 'new', 'nod', 
    'not', 'now', 'nut', 'oak', 'odd', 'off', 'oil', 'old', 'one', 'our', 'out', 
    'owl', 'pad', 'pan', 'pat', 'paw', 'pay', 'pen', 'pet', 'pie', 'pig', 'pin', 
    'pit', 'pop', 'pot', 'put', 'rag', 'ram', 'ran', 'rat', 'red', 'rib', 'rid', 
    'rig', 'rim', 'rip', 'rod', 'rot', 'row', 'rug', 'run', 'sad', 'sag', 'sap', 
    'sat', 'saw', 'say', 'sea', 'see', 'set', 'sew', 'she', 'shy', 'sip', 'sit', 
    'sky', 'sob', 'son', 'sow', 'sun', 'tab', 'tan', 'tap', 'tar', 'tax', 'tea', 
    'tie', 'tin', 'tip', 'top', 'toy', 'try', 'tub', 'two', 'use', 'van', 'vet', 
    'vow', 'war', 'was', 'wax', 'way', 'web', 'wet', 'who', 'why', 'wig', 'win', 
    'wow', 'yes', 'yet', 'you', 'zoo'
    ];

    type foundWordType = {
        [key: string]: number[][];
    }

    function findAllNewWords(startRow: number, startCol: number) {
        let foundWords: foundWordType = {};
        for (let s = 0; s <= startCol; s++){
            let currentWord = '';
            for (let i = s; i <= cols - 1; i++) {
                if (gameGrid[startRow][i] === '.') break;
                currentWord += gameGrid[startRow][i];
                if (i < startCol) continue;
                console.log("currentWord: ", currentWord);
                if (wordList.includes(currentWord.toLowerCase())) {
                    console.log('found a valid Word!', currentWord);
                    foundWords[currentWord] = [];
                    for (let j = startCol; j <= i; j++) {
                        foundWords[currentWord].push([startRow, j])
                    }
                    
                } 
            }
        }
        return foundWords;
    }
</script>
<body class="w-full h-screen max-h-screen bg-slate-500">

    <h1 class="text-center text-4xl pt-8 text-blue-200 w-2/4 m-auto">Letter Game</h1>
    <button on:click={startGame} class="block text-center text-4xl px-2 text-black bg-blue-200 mx-auto border border-black">Start</button>
    <div id="gameBox" class="relative top-[10px] bg-slate-900 max-w-[540px] w-full h-[600px] m-auto text-6xl outline outline-4 outline-slate-300">

    </div>
    {#if !gameOn}
    <div class="absolute top-[300px] left-2/4 -translate-x-2/4 border border-black bg-red-500 text-black text-4xl px-4">Game Over</div>
    {/if}
</body>
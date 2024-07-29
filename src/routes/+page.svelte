<script lang="ts">
    import { onDestroy, onMount } from "svelte";
    // import words from 'an-array-of-english-words';
    import { fellowText } from "$lib/fellowText";


    const fellowWords = fellowText.split(' ');
    let uniqueWords: string[] = [];
    for (let word of fellowWords) {
        if (uniqueWords.includes(word.toLowerCase())) {
            continue;
        } else {
            uniqueWords.push(word.toLowerCase());
        }
    }

    // function getLetterCounts() {
    //     let letters = []
    //     for (let word of words) {
    //         if (word.length < 6) {
    //             for (let letter of word) {
    //                 letters.push(letter)
    //             }
    //         }
    //     }
    //     let letterCounts: Record<string, number> = {};
    //     for (let alpha of alphabet) {
    //         letterCounts[alpha.toLowerCase()] = 0;
    //     }
    //     for (let letter of letters) {
    //         letterCounts[letter.toLowerCase()] += 1;
    //     }
    //     console.log(letterCounts);
    //     return letterCounts;
    // }

    // const letterCounts: Record<string, number> = {a:195832, b:46410, c: 102513,d: 83935,e: 286167,f: 29721,g: 69952,h: 63210,i: 231005,j: 4102,k: 22654,l: 132826,
    //     m: 73474,n: 171212,o: 169403,p: 76416,q: 4200,r: 176849,s: 244618,t: 166977,u: 83479,v: 23437,w:18657,x: 7098 ,y: 41314,z: 12299
    //  }


    const shortLetterCounts: Record<string, number> = {
        a:8387, b: 2317,c: 2657,d: 3388,e: 8980,f: 1677,g: 2381,h: 2481,i: 5196,j: 458,k: 2225,l: 4563
        ,m: 2817,n: 4037,o: 6418,p: 2887,q: 136,r: 5436,s: 8604,t: 4551,u: 3595,v: 954,w: 1606,x: 412,y: 2780,z: 628
    }

    function sumArray(numbers: number[]): number {
        return numbers.reduce((accumulator, currentValue) => accumulator + currentValue, 0);
    }
    
    function letterProbs() {
        let allProbs: Record<string, number> = {};
        const allCounts = Object.values(shortLetterCounts);
        const sumCounts = sumArray(allCounts);
        let totalProbs = 0;
        for (let letterKey of Object.keys(shortLetterCounts)) {
            totalProbs += shortLetterCounts[letterKey] / sumCounts;
            allProbs[letterKey] = totalProbs;
        }
        console.log(allProbs);
        return allProbs;
    }

    const alphaProbs = letterProbs();

    function getRandomLetter() {
        const rand = Math.random();
        for (let letterKey of Object.keys(alphaProbs)) {
            if (rand < alphaProbs[letterKey]) {
                return letterKey.toUpperCase();
            }
        }
        return 'z'.toUpperCase();
    }

    // const activeLetter = document.getElementById("activeLetter");
    // activeLetter?.style.left
    let fresh = true;
    let x = 0;
    let y = 0;
    const step = 60;
    const letterSize = 60;
    const gameWidth = 540;
    const gameHeight = 600;
    const rows = (gameHeight / letterSize);
    const cols = (gameWidth / letterSize);
    let activePosition: number[] = [];
    let activeLetterText = '';
    let lastMove = 0;
    let gameSpeed = 10;
    let gameInterval: ReturnType<typeof setInterval>;
    let gameGrid: string[][] = [];
    let columnDepths: number[] = [];
    let currentColumn = 0;
    let gameOn = false;
    let letterDivGrid: (HTMLDivElement | null)[][] = [];
    let activeLetter: HTMLDivElement;
    let nextLetters: string[] = [];

    function buildNextLetters() {
        for (let i = 0; i < 6; i++) {
            const randomLetter = getRandomLetter();
            nextLetters.push(randomLetter);
        }
    }

    function updateNextLetters() {
        let newNextLetters = [];
        for (let i = 0; i < 5; i++) {
            newNextLetters.push(nextLetters[i + 1])
        }
        const randomLetter = getRandomLetter();
        newNextLetters.push(randomLetter);
        nextLetters = newNextLetters;
    }

    function updateNextDivs() {
        for (let i = 0; i < 6; i++) {
            const thisDiv = document.getElementById(`next${i + 1}`);
            if (thisDiv) {
                thisDiv.textContent = nextLetters[i];
            }
        }
    }

    function updateColumnDepths() {
        columnDepths = [];
        for (let col in gameGrid[0]) {
            let colCount = 0;
            for (let row in gameGrid) {
                if (gameGrid[row][col] === '.') {
                    colCount += 1;
                } else {
                    break;
                }
            }
        columnDepths.push(colCount);
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

    function buildDivGrid() {
        letterDivGrid = [];
        for (let i = 0; i < rows; i++) {
            let currentRow = [];
            for (let j = 0; j < cols; j++) {
                currentRow.push(null)
            }
            letterDivGrid.push(currentRow);
        }
    }

    const alphabet: string[] = [
        'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M',
        'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z'
    ];


// Function to handle div creation
    const createActiveLetter = () => {
        
        activePosition = [4 * letterSize, 0];
        currentColumn = 4;
        const gameBox = document.getElementById('gameBox');
        activeLetter = document.createElement('div');
        activeLetterText = nextLetters[0];
        activeLetter.textContent = activeLetterText;
        activeLetter.style.position = 'absolute';
        activeLetter.style.backgroundColor = '#fef9c3';
        activeLetter.style.width = `${letterSize}px`;
        activeLetter.style.height = `${letterSize}px`;
        activeLetter.style.top = `${activePosition[1]}px`;
        activeLetter.style.left = `${activePosition[0]}px`;
        activeLetter.style.border = 'black 1px solid';
        activeLetter.style.textAlign = 'center';
        if (gameBox) gameBox.appendChild(activeLetter);
    };

    const handleKeyDown = (event: KeyboardEvent) => {
        if (gameOn) {

            switch (event.key) {
                case "ArrowUp":
                    break;
                case "ArrowDown":
                    if(activePosition[1]< (columnDepths[currentColumn] -1) * letterSize) {
                        activePosition[1] = (columnDepths[currentColumn] -1) * letterSize;
                        activeLetter.style.top = `${activePosition[1]}px`;
                        // columnDepths[currentColumn] -= 1;
                    }
                    break;
                case "ArrowLeft":
                    if(activePosition[0] > 0) {
                        activePosition[0] -= step;
                        activeLetter.style.left = `${activePosition[0]}px`;
                        currentColumn -= 1;
                    }
                    break;
                case "ArrowRight":
                    if (activePosition[0]< gameWidth - letterSize) {
                        activePosition[0] += step;
                        activeLetter.style.left = `${activePosition[0]}px`;
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
        // getLetterCounts();
        gameOn = true;
        fresh = false;
        buildNextLetters();
        updateNextLetters();
        console.log('about to update divs: ', nextLetters);
        updateNextDivs();
        const gameBox = document.getElementById('gameBox');
        if (gameBox) gameBox.innerHTML = '';
        buildGameGrid();
        buildDivGrid();
        updateColumnDepths();
        console.log(columnDepths);
        console.log(gameGrid);
        createActiveLetter();
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
        console.log(letterProbs());
        clearInterval(gameInterval);
        gameInterval = setInterval(() => {
            if (!checkColumnDepths()) {
                gameOn = false;
                clearInterval(gameInterval);
            }
            // check if enough time has passed since last interval, based on game speed.
            if (Date.now() > lastMove + gameSpeed) {
                // check if the current letter is at the bottom of a column
                if (activePosition[1] < (columnDepths[currentColumn] -1) * letterSize) {
                    activePosition[1] += 1;
                    activeLetter.style.top = `${activePosition[1]}px`;
                    lastMove = Date.now();
                } else {
                    // add to the gamegrid and switch to a new letter
                    const activeRow = Math.round(activePosition[1] / letterSize);
                    const activeCol = Math.round(activePosition[0] / letterSize);
                    const copiedLetterDiv = activeLetter.cloneNode(true) as HTMLDivElement;
                    letterDivGrid[activeRow][activeCol] = copiedLetterDiv;
                    const gameBox = document.getElementById('gameBox');
                    if (gameBox) gameBox.appendChild(copiedLetterDiv);
                    activeLetter.remove();
                    gameGrid[columnDepths[currentColumn] - 1][currentColumn] = activeLetterText;
                    const foundNewWords = findAllNewWords(columnDepths[currentColumn] - 1, currentColumn);
                    columnDepths[currentColumn] -= 1;
                    const redLetters = colorWordLetters(foundNewWords);
                    if (redLetters.length > 0) {
                        console.log("words: ", foundNewWords);
                        updateBoard(redLetters);
                    }
                    createActiveLetter();
                    updateNextLetters();
                    updateNextDivs();
                    console.log(letterDivGrid);
                    console.log(gameGrid);
                    lastMove = Date.now();
                }
            }
        }, 10)
    }

    type foundWordType = {
        [key: string]: number[][];
    }

    function colorWordLetters(foundWords: foundWordType) {
        let redLetterCoords = [];
        for (let key of Object.keys(foundWords)) {
            const wordCoords = foundWords[key];
            for (let coords of wordCoords) {
                redLetterCoords.push([coords[0], coords[1]]);
                const thisDiv = letterDivGrid[coords[0]][coords[1]];
                if (thisDiv) thisDiv.style.backgroundColor = 'orange';

            }
        }
        return redLetterCoords;
    }

    // Function to check if two arrays are equal
    function arraysEqual(arr1: number[], arr2: number[]): boolean {
    if (arr1.length !== arr2.length) {
        return false;
    }
    for (let i = 0; i < arr1.length; i++) {
        if (arr1[i] !== arr2[i]) {
        return false;
        }
    }
    return true;
    }


    function findAllNewWords(startRow: number, startCol: number) {
        let foundWords: foundWordType = {};
        // find forward words
        for (let s = 0; s <= startCol; s++){
            let currentWord = '';
            for (let i = s; i <= cols - 1; i++) {
                if (gameGrid[startRow][i] === '.') break;
                currentWord += gameGrid[startRow][i];
                if (i < startCol) continue;
                if (currentWord.length > 2 && uniqueWords.includes(currentWord.toLowerCase())) {
                    console.log('found a valid Word!', currentWord);
                    foundWords[currentWord] = [];
                    for (let j = s; j <= i; j++) {
                        foundWords[currentWord].push([startRow, j])
                    }
                    
                } 
            }
        }
        // find downward words
        for (let s = 0; s <= startRow; s++){
            let currentWord = '';
            for (let i = s; i <= rows - 1; i++) {
                if (gameGrid[i][startCol] === '.') break;
                currentWord += gameGrid[i][startCol];
                if (i < startRow) continue;
                if (currentWord.length > 2 && uniqueWords.includes(currentWord.toLowerCase())) {
                    console.log('found a valid Word!', currentWord);
                    foundWords[currentWord] = [];
                    for (let j = s; j <= i; j++) {
                        foundWords[currentWord].push([j, startCol])
                    }
                    
                } 
            }
        }


        return foundWords;
    }

    function countDeletesUnderLetter(grid: string[][]) {
        let dropGrid: number[][] = [];
        for (let row in grid) {
            let newRow = [];
            for (let col in grid[row]) {
                if (grid[row][col] !== '.') {
                    if (grid[row][col] !== 'del') {
                        let count = 0;
                        for (let i = parseInt(row) + 1; i < rows; i++) {
                            if (grid[i][col] == 'del') {
                                count += 1;
                            }
                        }
                        newRow.push(count);
                    } else {
                        newRow.push(-1);
                    }

                } else {
                    newRow.push(0);
                }
            }
            dropGrid.push(newRow)
        }
        return dropGrid;
    }

    function addDeletesToGrid(coords: number[][]) {
        for (let coord of coords) {
            gameGrid[coord[0]][coord[1]] = 'del';
        }
    }

    function dropLetters(dropGrid: number[][]) {
        console.log("drop grid: ", dropGrid);
        let droppedGrid:string[][] = JSON.parse(JSON.stringify(gameGrid));
        for (let r = dropGrid.length -1; r >= 0; r--) {
            for (let c = 0; c < dropGrid[r].length; c++) {
                if (dropGrid[r][c] !== 0) {
                    // check if this is a cell where letters should drop
                    if (dropGrid[r][c] > 0) {
                        // update the gameGrid
                        const drops = dropGrid[r][c];
                        droppedGrid[r + drops][c] = gameGrid[r][c];
                        droppedGrid[r][c] = '.';

                        // update the actual divGrid
                        const newTopLocation = r * letterSize + dropGrid[r][c] * letterSize;
                        const thisDiv = letterDivGrid[r][c];
                        if (thisDiv) {
                            const deepCopiedDiv = thisDiv.cloneNode(true) as HTMLDivElement;
                            const gameBox = document.getElementById('gameBox');
                            if (gameBox) gameBox.appendChild(deepCopiedDiv);
                            deepCopiedDiv.style.top = `${newTopLocation}px`;
                            letterDivGrid[r + drops][c] = deepCopiedDiv;
                            thisDiv.remove();
                            letterDivGrid[r][c] = null;
                        }

                    } else {
                        droppedGrid[r][c] = '.';
                        const thisDiv = letterDivGrid[r][c];
                        if (thisDiv) {
                            thisDiv.remove()
                            letterDivGrid[r][c] = null;
                        }
                    }
                    // these are cells where letters should be deleted

                }
            }
        }
        gameGrid = droppedGrid;
    }

    function updateBoard(indexes: number[][]) {
        addDeletesToGrid(indexes);
        console.log("delete grid:", gameGrid);
        const dropGrid = countDeletesUnderLetter(gameGrid);
        dropLetters(dropGrid);
        updateColumnDepths();
    }
</script>
<body class="w-screen max-w-[800px] mx-auto h-screen max-h-screen bg-green-950 bg-opacity-60">

    <h1 class="text-center text-4xl pt-8 text-blue-200 w-2/4 m-auto">Letter Game</h1>
    <button on:click={startGame} class="block text-center text-4xl px-2 text-black bg-blue-200 mx-auto border border-black rounded-lg hover:bg-blue-300 active:bg-blue-400">Start</button>
    <div id="gameBoxContainer" class="relative inline-flex top-[10px] max-w-[800px] w-full h-[600px] text-6xl">
        <div id="nextBox" class="relative ml-4 flex flex-col justify-between bg-slate-900 w-[80px] h-[600px] text-6xl py-4 outline outline-4 outline-slate-300">
        </div>
        <div id="gameBox" class="relative bg-slate-900 min-w-[540px] max-w-[540px] mx-auto w-full h-[600px] text-6xl outline outline-4 outline-slate-300">
            
        </div>
        <div id="nextBox" class="relative flex flex-col justify-between bg-slate-900 w-[80px] h-[600px] mr-4 text-6xl py-4 outline outline-4 outline-slate-300">

            <div id="next1" class= "w-[60px] h-[60px] bg-yellow-100 bg-opacity-80 mx-auto text-center"></div>
            <div id="next2" class= "w-[60px] h-[60px] bg-yellow-100 bg-opacity-70 mx-auto text-center"></div>
            <div id="next3" class= "w-[60px] h-[60px] bg-yellow-100 bg-opacity-60 mx-auto text-center"></div>
            <div id="next4" class= "w-[60px] h-[60px] bg-yellow-100 bg-opacity-50 mx-auto text-center"></div>
            <div id="next5" class= "w-[60px] h-[60px] bg-yellow-100 bg-opacity-40 mx-auto text-center"></div>
            <div id="next6" class= "w-[60px] h-[60px] bg-yellow-100 bg-opacity-30 mx-auto text-center"></div>

        </div>
    </div>
    {#if !gameOn && !fresh}
    <div class="absolute top-[300px] left-2/4 -translate-x-2/4 border border-black bg-red-500 text-black text-4xl px-4">Game Over</div>
    {/if}
</body>
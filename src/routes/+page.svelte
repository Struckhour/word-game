<script lang="ts">
    import { onDestroy, onMount } from "svelte";
    // import words from 'an-array-of-english-words';

    import lordWords from '$lib/lordWords.json';
    import lordOnlyWords from '$lib/lordOnlyWords.json'
  import { fly } from "svelte/transition";


    // let uniqueWords: string[] = [];
    // for (let word of fellowWords) {
    //     if (uniqueWords.includes(word.toLowerCase())) {
    //         continue;
    //     } else {
    //         uniqueWords.push(word.toLowerCase());
    //     }
    // }
    // for (let word of towerWords) {
    //     if (uniqueWords.includes(word.toLowerCase())) {
    //         continue;
    //     } else {
    //         uniqueWords.push(word.toLowerCase());
    //     }
    // };
    // for (let word of returnWords) {
    //     if (uniqueWords.includes(word.toLowerCase())) {
    //         continue;
    //     } else {
    //         uniqueWords.push(word.toLowerCase());
    //     }
    // };


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

    const vowelMultiplier = 1.5;
    const shortLetterCounts: Record<string, number> = {
        a:8387*vowelMultiplier, b: 2317,c: 2657,d: 3388,e: 8980*vowelMultiplier,f: 1677,g: 2381,h: 2481,i: 5196*vowelMultiplier,j: 458,k: 2225,l: 4563
        ,m: 2817,n: 4037,o: 6418*vowelMultiplier,p: 2887,q: 136,r: 5436,s: 8604,t: 4551,u: 3595*vowelMultiplier,v: 954,w: 1606,x: 412,y: 2780,z: 628
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
    const step = 11.11;
    const letterWidth = 11.11;
    const letterHeight = 9.09;
    const rows = 11;
    const cols = 9;
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
    let gameOver = false;
    let allFoundWords: string[] = [];
    let foundWordDivs: HTMLDivElement[] = [];
    let score = 0;
    let showMenu = false;


    function buildNextLetters() {
        for (let wordDiv of foundWordDivs) {
            wordDiv.remove();
        }
        for (let i = 0; i < 8; i++) {
            const randomLetter = getRandomLetter();
            nextLetters.push(randomLetter);
        }
    }

    function updateNextLetters() {
        let newNextLetters = [];
        for (let i = 0; i < 7; i++) {
            newNextLetters.push(nextLetters[i + 1])
        }
        const randomLetter = getRandomLetter();
        newNextLetters.push(randomLetter);
        nextLetters = newNextLetters;
    }

    function updateNextDivs() {
        for (let i = 0; i < 8; i++) {
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

    function deleteBoardDivs() {
        for (let divRow of letterDivGrid) {
            for (let letterDiv of divRow) {
                if (letterDiv) letterDiv.remove();
            }
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
        
        activePosition = [4 * letterWidth, 0];
        currentColumn = 4;
        const gameBox = document.getElementById('gameBox');
        activeLetter = document.createElement('div');
        activeLetterText = nextLetters[0];
        activeLetter.textContent = activeLetterText;
        activeLetter.style.position = 'absolute';
        activeLetter.style.backgroundColor = '#fef9c3';
        activeLetter.style.width = `${letterWidth}%`;
        activeLetter.style.height = `${letterHeight}%`;
        activeLetter.style.top = `${activePosition[1]}%`;
        activeLetter.style.left = `${activePosition[0]}%`;
        activeLetter.style.border = 'black 1px solid';
        activeLetter.style.textAlign = 'center';
        activeLetter.style.display = 'flex';
        activeLetter.style.alignItems = 'center';
        activeLetter.style.justifyContent = 'center';
        if (gameBox) gameBox.appendChild(activeLetter);
    };

    const handleKeyDown = (event: KeyboardEvent) => {
        if (gameOn) {

            switch (event.key) {
                case "ArrowUp":
                    break;
                case "ArrowDown":
                    if(activePosition[1]< (columnDepths[currentColumn] -1) * letterHeight) {
                        activePosition[1] = (columnDepths[currentColumn] -1) * letterHeight;
                        activeLetter.style.top = `${activePosition[1]}%`;
                        // columnDepths[currentColumn] -= 1;
                    }
                    break;
                case "ArrowLeft":
                    if(activePosition[0] > 0 && (activePosition[1] < (columnDepths[currentColumn -1] -1) * letterHeight)  ) {
                        activePosition[0] -= step;
                        activeLetter.style.left = `${activePosition[0]}%`;
                        currentColumn -= 1;
                    }
                    break;
                case "ArrowRight":
                    if (activePosition[0]< (cols-1) * letterWidth && (activePosition[1] < (columnDepths[currentColumn + 1] -1) * letterHeight)) {
                        activePosition[0] += step;
                        activeLetter.style.left = `${activePosition[0]}%`;
                        currentColumn += 1;
                    }
                    break;
            }
        }
    };

    function leftClick() {
        if(activePosition[0] > 0 && (activePosition[1] < (columnDepths[currentColumn -1] -1) * letterHeight)  ) {
            activePosition[0] -= step;
            activeLetter.style.left = `${activePosition[0]}%`;
            currentColumn -= 1;
        }
    }

    function downClick() {
        if(activePosition[1]< (columnDepths[currentColumn] -1) * letterHeight) {
            activePosition[1] = (columnDepths[currentColumn] -1) * letterHeight;
            activeLetter.style.top = `${activePosition[1]}%`;
                        // columnDepths[currentColumn] -= 1;
        }
    }

    function rightClick() {
        if (activePosition[0]< (cols-1) * letterWidth && (activePosition[1] < (columnDepths[currentColumn + 1] -1) * letterHeight)) {
            activePosition[0] += step;
            activeLetter.style.left = `${activePosition[0]}%`;
            currentColumn += 1;
        }
    }

    onMount(() => {

        async function lockOrientation() {
            try {
                const screenOrientation = (screen.orientation) as any;
                if (screenOrientation.lock) {
                await screenOrientation.lock("portrait");
                } else {
                console.warn("Orientation lock API is not supported.");
                }
            } catch (err) {
                console.error("Orientation lock failed:", err);
            }
        }

        lockOrientation();
        window.addEventListener("keydown", handleKeyDown);
        return () => {
            window.removeEventListener("keydown", handleKeyDown);
        if (screen.orientation && screen.orientation.unlock) {
            screen.orientation.unlock();
        } else if (window.screen && window.screen.orientation && window.screen.orientation.unlock) {
            window.screen.orientation.unlock();
        }
        };

    });

    onDestroy(() => {
        clearInterval(gameInterval);
    })

    function pauseGame() {
        if (gameOn && !gameOver) {
            gameOn = false;
            clearInterval(gameInterval);
            return;
        }
    }

    function startGame() {
        if (gameOn && !gameOver) {
            gameOn = false;
            clearInterval(gameInterval);
            return;
        }
        if (!gameOn && !gameOver && !fresh) {
            gameOn = true;
            resetGame();
            return;
        }
        // getLetterCounts();
        gameOn = true;
        fresh = false;
        gameOver = false;
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

    function setGameSpeed() {
        if (score > 100) {
            gameSpeed = score / 10;
        }
    }

    function resetGame() {
        console.log(letterProbs());
        clearInterval(gameInterval);
        gameInterval = setInterval(() => {
            if (!checkColumnDepths()) {
                gameOn = false;
                gameOver = true;
                clearInterval(gameInterval);
            }
            // check if enough time has passed since last interval, based on game speed.
            if (Date.now() > lastMove + 10) {
                // check if the current letter is at the bottom of a column
                if (activePosition[1] < (columnDepths[currentColumn] -1) * letterHeight) {
                    activePosition[1] += .2 * gameSpeed / 10;
                    activeLetter.style.top = `${activePosition[1]}%`;
                    lastMove = Date.now();
                } else {
                    // add to the gamegrid and switch to a new letter
                    const activeRow = Math.round(activePosition[1] / letterHeight);
                    const activeCol = Math.round(activePosition[0] / letterWidth);
                    const copiedLetterDiv = activeLetter.cloneNode(true) as HTMLDivElement;
                    copiedLetterDiv.style.top = `${activeRow * letterHeight}%`;
                    letterDivGrid[activeRow][activeCol] = copiedLetterDiv;
                    const gameBox = document.getElementById('gameBox');
                    if (gameBox) gameBox.appendChild(copiedLetterDiv);
                    activeLetter.remove();
                    gameGrid[columnDepths[currentColumn] - 1][currentColumn] = activeLetterText;

                    // check for words
                    const foundNewWords = findAllNewWords(columnDepths[currentColumn] - 1, currentColumn);
                    columnDepths[currentColumn] -= 1;
                    const redLetters = colorWordLetters(foundNewWords);
                    if (redLetters.length > 0) {
                        setGameSpeed();
                        addWordDivs(Object.keys(foundNewWords));
                        allFoundWords = allFoundWords.concat(Object.keys(foundNewWords));
                        updateScore(Object.keys(foundNewWords));
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

    function updateScore(wordList: string[]) {
        for (let word of wordList) {
            if (word.length === 3) {
                score += 3;
            } else if (word.length === 4) {
                score += 8;
            } else if (word.length === 5) {
                score += 18;
            } else if (word.length === 6) {
                score += 38;
            } else if (word.length > 6) {
                score += 68;
            }
        }
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
                if (currentWord.length > 2 && lordOnlyWords.includes(currentWord.toLowerCase())) {
                    score += 100;
                    deleteBoardDivs();
                    buildGameGrid();
                    buildDivGrid();
                } 
                if (currentWord.length > 2 && lordWords.includes(currentWord.toLowerCase())) {
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
                if (currentWord.length > 2 && lordWords.includes(currentWord.toLowerCase())) {
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

    function addWordDivs(newWords: string[]) {
        for (let word of newWords) {
            const pastWords = document.getElementById('pastWords');
            let wordDiv = document.createElement('div');
            wordDiv.textContent = word;
            foundWordDivs.push(wordDiv)
            if (foundWordDivs.length > 15) {
                foundWordDivs[0].remove();
                foundWordDivs.shift();
            }
            if (pastWords) pastWords.appendChild(wordDiv);
        }
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
                        const newTopLocation = r * letterHeight + dropGrid[r][c] * letterHeight;
                        const thisDiv = letterDivGrid[r][c];
                        if (thisDiv) {
                            const deepCopiedDiv = thisDiv.cloneNode(true) as HTMLDivElement;
                            const gameBox = document.getElementById('gameBox');
                            if (gameBox) gameBox.appendChild(deepCopiedDiv);
                            deepCopiedDiv.style.top = `${newTopLocation}%`;
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
<body class="overflow-hidden square relative mx-auto min-h-[90vh] bg-green-950 bg-opacity-60">
    {#if showMenu}
    <div transition:fly={{ x:400, duration:300}} class="overflow-hidden w-[350px] h-screen absolute right-0 bg-blue-950 bg-opacity-[98%] z-20 border-l-4 border-blue-100">
        <button on:click={() => {showMenu = false;}} class="text-4xl mt-12 ml-4 w-full border-y border-blue-200 hover:bg-blue-700 active:bg-blue-900 text-blue-100">Close --></button>
        <ul class="text-xl list-disc list-inside ml-4 mt-4 text-blue-200">
            <li>Use arrow keys or buttons below game board.</li>
            <li>Make words by lining up letters from left-to-right or top-to-bottom (like Scrabble).</li>
            <li>Words must be in the text of Lord of the Rings to be valid. This includes dialogue and the removal of accents or punctuation.</li>
            <li>Longer words are worth way more points.</li>
            <li>Words that are ONLY in Lord of the Rings (i.e. are not English words), such as FRODO, give 100 bonus points and clear the board.</li>
        </ul>
        <button on:click={() => {showMenu = false;}} class="text-4xl mt-12 ml-4 w-full border-y border-blue-200 hover:bg-blue-700 active:bg-blue-900 text-blue-100">--></button>
    </div>
    {/if}

    <div class="relative h-[5%] border-b border-black flex items-center justify-start text-left">
        <h1 class="ml-4 text-center text-xl font-serif text-blue-200 tracking-widest h-full pb-1 flex items-end">ENT-RIS</h1>
        <h3 class="ml-4 text-center text-sm text-blue-300 pb-1 flex items-end h-full">A Middle-Earth word-building game</h3>
    </div>
    
    <!--menu button row -->
    <div class="w-[91%] md:w-[71%] mx-auto grid grid-cols-3 text-center h-[8%] py-4 md:py-1">
        <div class="flex items-center text-left pl-2 text-blue-200 text-xl md:text-2xl">Score: {score}</div>
        {#if !gameOn && fresh && !gameOver}
        <button on:click={startGame} class="inline-block text-center text-xl md:text-4xl px-2 text-black bg-blue-300 mx-auto border border-black rounded-lg hover:bg-blue-200 active:bg-blue-400">Start</button>
        {:else if !gameOn && !fresh && !gameOver}
        <button on:click={startGame} class="inline-block text-center text-xl md:text-4xl px-2 text-black bg-blue-300 mx-auto border border-black rounded-lg hover:bg-blue-200 active:bg-blue-400">Resume</button>
        {:else if !gameOn && !fresh && gameOver}
        <button on:click={startGame} class="inline-block text-center text-xl md:text-4xl px-2 text-black bg-blue-300 mx-auto border border-black rounded-lg hover:bg-blue-200 active:bg-blue-400">Restart</button>
        {:else}
        <button on:click={startGame} class="inline-block text-center text-xl md:text-4xl px-2 text-black bg-blue-300 mx-auto border border-black rounded-lg hover:bg-blue-200 active:bg-blue-400">Pause</button>
        {/if}
        <button on:click={() => {showMenu = !showMenu; pauseGame()}} class="inline-block text-right text-xl md:text-4xl px-2 text-black bg-blue-300 mx-auto mr-2 border border-black rounded-lg hover:bg-blue-200 active:bg-blue-400">
            Rules
        </button>
    </div>
    <div id="gameBoxContainer" class="relative inline-flex max-w-[800px] w-full h-[75%] text-6xl left-2/4 -translate-x-2/4">
        <div id="pastWords" class="overflow-hidden flex flex-col-reverse flex-grow mx-2 relative justify-start bg-slate-900 w-[9%] h-full text-xs sm:text-base tracking-tighter text-slate-300 py-4 outline outline-2 outline-slate-300">
            <!-- <div class="">wordword</div>
            <div class="">word2</div> -->
        </div>
        <div id="gameBox" class="relative bg-slate-900 w-[71%] mx-auto h-full text-2xl md:text-4xl lg:text-6xl outline outline-2 outline-slate-300">
            
        </div>
        <div id="nextBox" class="relative flex flex-grow flex-col justify-between bg-slate-900 w-[9%] h-full mx-2 text-2xl sm:text-4xl md:text-6xl py-2 outline outline-2 outline-slate-300">
            <div id="next1" class= "flex items-center justify-center max-h-[10%] max-w-[100%] aspect-square bg-yellow-100 bg-opacity-80 mx-auto"></div>
            <div id="next2" class= "flex items-center justify-center max-h-[10%] max-w-[100%] aspect-square bg-yellow-100 bg-opacity-70 mx-auto"></div>
            <div id="next3" class= "flex items-center justify-center max-h-[10%] max-w-[100%] aspect-square bg-yellow-100 bg-opacity-60 mx-auto"></div>
            <div id="next4" class= "flex items-center justify-center max-h-[10%] max-w-[100%] aspect-square bg-yellow-100 bg-opacity-50 mx-auto"></div>
            <div id="next5" class= "flex items-center justify-center max-h-[10%] max-w-[100%] aspect-square bg-yellow-100 bg-opacity-40 mx-auto"></div>
            <div id="next6" class= "flex items-center justify-center max-h-[10%] max-w-[100%] aspect-square bg-yellow-100 bg-opacity-30 mx-auto"></div>
            <div id="next7" class= "flex items-center justify-center max-h-[10%] max-w-[100%] aspect-square bg-yellow-100 bg-opacity-30 mx-auto"></div>
            <div id="next8" class= "flex items-center justify-center max-h-[10%] max-w-[100%] aspect-square bg-yellow-100 bg-opacity-30 mx-auto"></div>
        </div>
    </div>
    <!-- left right down buttons -->
    <div class="grid grid-cols-3 grid-rows-1 w-[71%] gap-x-4 max-w-[540px] mx-auto xl:hidden h-[10%] py-4">
        <button on:click={leftClick} class="bg-green-600 rounded-2xl text-2xl h-full active:bg-green-400 border border-white">left</button>
        <button on:click={downClick} class="bg-green-600 rounded-2xl text-2xl h-full active:bg-green-400 border border-white">down</button>
        <button on:click={rightClick} class="bg-green-600 rounded-2xl text-2xl h-full active:bg-green-400 border border-white">right</button>
    </div>
    {#if gameOver && score < 200}
    <div class="absolute top-[300px] left-2/4 -translate-x-2/4 border border-black bg-red-500 text-black text-4xl px-4">Game Over</div>
    {:else if gameOver && score >=200}
    <div class="absolute top-2/4 -translate-y-2/4 left-2/4 p-2 -translate-x-2/4 border border-black bg-yellow-500 text-black text-lg">
        <h2 class="text-xl text-center font-bold">Suspicious Activity Detected</h2>
        Our system has detected possible cheating. A company representative has been dispatched to the IP affiliated with this account: [North Okanagan Youth and Family Services]. If, after thorough inspection, Agent B. Childress confirms your full compliance, no further action will be required. Otherwise you will be rigorousl>>></div>
    {/if}
</body>

<style>
    html, body {
      touch-action: manipulation;
    }
    .square {
      width: 100vw;
      height: 90vh;
    }

    @media (min-width: 800px) {
      .square {
        max-width: 800px;
        width: 100vw;
        height: 95vh;
      }
    }

</style>
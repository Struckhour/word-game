<script lang="ts">
    import { onDestroy, onMount } from "svelte";
    import words from 'an-array-of-english-words';


    // function getLetterCounts() {
    //     let letters = []
    //     for (let word of words) {
    //         for (let letter of word) {
    //             letters.push(letter)
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

    const letterCounts: Record<string, number> = {a:195832, b:46410, c: 102513,d: 83935,e: 286167,f: 29721,g: 69952,h: 63210,i: 231005,j: 4102,k: 22654,l: 132826
                        ,m: 73474,n: 171212,o: 169403,p: 76416,q: 4200,r: 176849,s: 244618,t: 166977,u: 83479,v: 23437,w:18657,x: 7098 ,y: 41314,z: 12299}

    function sumArray(numbers: number[]): number {
        return numbers.reduce((accumulator, currentValue) => accumulator + currentValue, 0);
    }
    
    function letterProbs() {
        let allProbs: Record<string, number> = {};
        const allCounts = Object.values(letterCounts);
        const sumCounts = sumArray(allCounts);
        let totalProbs = 0;
        for (let letterKey of Object.keys(letterCounts)) {
            totalProbs += letterCounts[letterKey] / sumCounts;
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
    let activeLetter = 0;
    let activeLetterText = '';
    let lastMove = 0;
    let gameSpeed = 10;
    let gameInterval: ReturnType<typeof setInterval>;
    let gameGrid: string[][] = [];
    let columnDepths: number[] = [];
    let currentColumn = 0;
    let gameOn = false;


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
        activeLetterText = getRandomLetter();
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
        fresh = false;
        gameOn = true;
        const gameBox = document.getElementById('gameBox');
        if (gameBox) gameBox.innerHTML = '';
        buildGameGrid();
        updateColumnDepths();
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
                if (letterLocs[activeLetter][1] < (columnDepths[currentColumn] -1) * letterSize) {
                    letterLocs[activeLetter][1] += 1;
                    letters[activeLetter].style.top = `${letterLocs[activeLetter][1]}px`;
                    lastMove = Date.now();
                } else {
                    // add to the gamegrid and switch to a new letter
                    gameGrid[columnDepths[currentColumn] - 1][currentColumn] = activeLetterText;
                    const foundNewWords = findAllNewWords(columnDepths[currentColumn] - 1, currentColumn);
                    columnDepths[currentColumn] -= 1;
                    console.log("words: ", foundNewWords);
                    const redLetters = colorWordLetters(foundNewWords);
                    if (redLetters.length > 0) {
                        updateBoard(redLetters);
                    }
                    createLetter();
                    lastMove = Date.now();
                }
            }
        }, 10)
    }

    type foundWordType = {
        [key: string]: number[][];
    }

    function colorWordLetters(foundWords: foundWordType) {
        let redLetters = [];
        let allCoords = []
        for (let key of Object.keys(foundWords)) {
            const wordCoords = foundWords[key];
            for (let coords of wordCoords) {
                for (let i in letterLocs) {
                    if (arraysEqual(letterLocs[i], coords)) {
                        redLetters.push(parseInt(i));
                        letters[i].style.backgroundColor = 'orange';
                    }
                }
            }
        }
        return redLetters.sort((a, b) => b - a);
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

// FIX THE REDLETTER FUNCTION SO THAT IT ONLY HAS ONE OF EACH INDEX TO DELETE
// 
// 
// ###########################################################################################################333




// ###########################################################################################################333









// ###########################################################################################################333



















// ###########################################################################################################333





















// ###########################################################################################################333
    function findAllNewWords(startRow: number, startCol: number) {
        let foundWords: foundWordType = {};
        for (let s = 0; s <= startCol; s++){
            let currentWord = '';
            for (let i = s; i <= cols - 1; i++) {
                if (gameGrid[startRow][i] === '.') break;
                currentWord += gameGrid[startRow][i];
                if (i < startCol) continue;
                if (currentWord.length > 2 && words.includes(currentWord.toLowerCase())) {
                    console.log('found a valid Word!', currentWord);
                    foundWords[currentWord] = [];
                    for (let j = s; j <= i; j++) {
                        foundWords[currentWord].push([j * letterSize, startRow * letterSize])
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

    function addDeletesToGrid(indexes: number[]) {
        let newGrid: string[][] = [];
        for (let row in gameGrid) {
            let newRow: string[] = [];
            for (let col in gameGrid[row]) {
                let match = false;
                for (let i of indexes) {
                    if (arraysEqual(letterLocs[i], [parseInt(col) * letterSize, parseInt(row) * letterSize])) {
                        newRow.push('del');
                        match = true;
                        break;
                    }     
                }
                if (!match) newRow.push(gameGrid[parseInt(row)][parseInt(col)]);
            }
            newGrid.push(newRow);
        }
        return newGrid;
    }

    function dropLetters(dropGrid: number[][]) {
        console.log("drop grid: ", dropGrid);
        let droppedGrid:string[][] = JSON.parse(JSON.stringify(gameGrid));
        for (let r = dropGrid.length -1; r >= 0; r--) {
            for (let c = 0; c < dropGrid[r].length; c++) {
                if (dropGrid[r][c] !== 0) {
                    // check if this is a cell where letters should drop
                    if (dropGrid[r][c] > 0) {
                        const drops = dropGrid[r][c];
                        droppedGrid[r + drops][c] = gameGrid[r][c];
                        droppedGrid[r][c] = '.';
                        for (let i in letterLocs) {
                            if (arraysEqual(letterLocs[i], [c * letterSize, r * letterSize])) {
                                const newTopLocation = r * letterSize + dropGrid[r][c] * letterSize;
                                letters[i].style.top = `${newTopLocation}px`;
                            }
                        }
                    // these are cells where letters should be deleted
                    } else {
                        droppedGrid[r][c] = '.';
                    }
                }
            }
        }
        gameGrid = droppedGrid;
        console.log("gameGrid after dropping: ", gameGrid);
    }

    function updateBoard(indexes: number[]) {
        const deleteGrid = addDeletesToGrid(indexes);
        console.log("delete grid:", deleteGrid);
        const dropGrid = countDeletesUnderLetter(deleteGrid);
        console.log(dropGrid);
        for (let index of indexes) {
            console.log("letter div about to be deleted", letters[index]);
            letters[index].remove();
        }
        dropLetters(dropGrid);
        updateColumnDepths();
    }
</script>
<body class="w-screen max-w-[800px] mx-auto h-screen max-h-screen bg-slate-500">

    <h1 class="text-center text-4xl pt-8 text-blue-200 w-2/4 m-auto">Letter Game</h1>
    <button on:click={startGame} class="block text-center text-4xl px-2 text-black bg-blue-200 mx-auto border border-black">Start</button>
    <div id="gameBox" class="relative top-[10px] bg-slate-900 max-w-[540px] w-full h-[600px] m-auto text-6xl outline outline-4 outline-slate-300">

    </div>
    {#if !gameOn && !fresh}
    <div class="absolute top-[300px] left-2/4 -translate-x-2/4 border border-black bg-red-500 text-black text-4xl px-4">Game Over</div>
    {/if}
</body>
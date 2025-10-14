# 🌀 Loopover Solver — Codewars 1 kyu

This project contains my solution to the **Loopover** challenge (1 kyu on Codewars).  
No AI could solve it — but I did 😄  

## 🧩 About the challenge
The goal is to reorder a scrambled grid into its solved state by shifting rows and columns, like a 2D Rubik’s Cube.  
👉 Try it yourself here: [loopover.xyz](https://loopover.xyz)

## 🧠 Logic behind my solution
The idea was to define a clear and deterministic process before coding:
1. Keep the **last column empty** to move numbers without breaking the order.
2. Solve the grid row by row, stopping before the last column and last row.
3. Then solve the **last column** using a similar method.
4. Handle **parity issues** on the last moves.
5. Finally, adjust until the grid matches the solved position.

Once this process was well defined, coding it was a piece of cake 🍰

Language : js / complexity : O(n²)


```js
function solve(mixedUpBoard, solvedBoard) {
  // Testing if the grid is solvable or not
  if(!isSolvable(mixedUpBoard, solvedBoard) && solvedBoard.length > 2) return null
  // Doing a translation where the right order is expressed in numbers
  let letterToNum = {}
  solvedBoard.flat().forEach((elem,i) => letterToNum[elem] = i+1)
  solvedBoard = solvedBoard.map(x => x.map(elem => elem = letterToNum[elem]))
  const rows = solvedBoard.length
  const cols = solvedBoard[0].length
  let res = []
  // Mapping to know where the characters can be located in O(1) instead of located them by looping in O(n^3)
  let position = {}
  for(let i = 0 ; i < rows ; i++){
    for(let j = 0 ; j < cols ; j++){
      let convertMix = letterToNum[mixedUpBoard[i][j]]
      position[convertMix] = {key : Number(convertMix), row : i , col : j}
    }
  }
  // Solving small grid
  if(solvedBoard.length === 2 && solvedBoard[0].length ===2){
    let count = 0
    while(!isMatching(position, solvedBoard) && count++<200){
      let random = Math.floor(Math.random() * 4)
      random === 0 ? moveRowLeft(0, position, res) : random === 1 ? moveRowRight(1, position, res) : random === 2 ? moveColDown(0, position, res) : moveColUp(1, position, res)
    }
    if(count>=200) return null
    return res
  }
  // Putting the characters at their right place except for them that belong to the last row and the last column
  for(let i = 0 ; i < rows -1 ; i++){
    for(let j = 0 ; j < cols -1 ; j++){
      if(position[solvedBoard[i][j]].row === i){
        if(solvedBoard[i][j] % cols === 1){
          while(position[solvedBoard[i][j]].col !== cols -2){
            moveRowRight(position[solvedBoard[i][j]].row, position, res)
          }
        } else {
          while(position[solvedBoard[i][j]].col !== cols -1){
            moveRowRight(position[solvedBoard[i][j]].row, position, res)
          }
          moveColDown(position[solvedBoard[i][j]].col, position, res)
          while(position[solvedBoard[i][j]-1].col !== cols -2){
            moveRowRight(position[solvedBoard[i][j-1]].row, position, res)
          }
          moveColUp(position[solvedBoard[i][j]].col, position, res)
          moveRowLeft(position[solvedBoard[i][j]].row, position, res)
        }
      } else{
        while(position[solvedBoard[i][j]].col < cols -1){
          moveRowRight(position[solvedBoard[i][j]].row, position, res)
        }
        while(position[solvedBoard[i][j]].row !== i){
          moveColUp(position[solvedBoard[i][j]].col, position, res)
        }
        moveRowLeft(position[solvedBoard[i][j]].row, position, res)
      }
    }
  }
  // Putting the characters of the last column in order except for the last two of them
  colNumInOrder(cols, rows, position, solvedBoard,res)
  if(isMatching(position, solvedBoard)) return res


  let rowEven = rows % 2 !== 0
  // Putting the characters from the last column in order
  while(!isAsc(position, solvedBoard) || (position[solvedBoard[0][cols-1]].row === 0 && position[solvedBoard[0][cols-1]].col === cols-1)){
    let bottomRow = getBottomRow(position, solvedBoard)
    let start = bottomRow.indexOf(solvedBoard[rows-1][0])
    let target = solvedBoard[rows-2][cols-1]
    let dontTouch = bottomRow.indexOf(target)
    if(dontTouch !== -1){
      bottomRow[dontTouch] = bottomRow[dontTouch-1]+1 || bottomRow[bottomRow.length-1]+1
    }
    for(let i = start+1 ; i <start + cols ; i++){
      if(bottomRow[i % cols] !== bottomRow[(i-1) % cols]+1){
        let nextNum = bottomRow[(i-1) % cols] +1
        let indNextNum = bottomRow.indexOf(nextNum)
        let numToReplace = bottomRow[i % cols]
        if(indNextNum !== -1){
          while(bottomRow[cols-1] !== nextNum){
            moveRowRight(rows-1, position, res)
            let lastNum = bottomRow.pop()
            bottomRow.unshift(lastNum)
          }
          bottomRow[cols-1] = numToExchange(rows-2, cols-1, position)
          moveColDown(cols-1, position, res)
          if(isAsc(position, solvedBoard)) break
          while(position[numToReplace].col !== cols-1){
            moveRowRight(rows-1, position, res)
            let lastNum = bottomRow.pop()
            bottomRow.unshift(lastNum)
          }
          bottomRow[cols-1] = nextNum
          moveColUp(cols-1, position, res)
        } else {
          while(position[numToReplace].col !== cols-1){
            moveRowRight(rows-1, position, res)
            let lastNum = bottomRow.pop()
            bottomRow.unshift(lastNum)
          }
          bottomRow[cols-1] = nextNum
          moveColDown(cols-1, position, res)
          if(isAsc(position, solvedBoard)) break
          if(nextNum !== solvedBoard[rows-1][cols-1] || (position[nextNum+1].col === cols-1 && position[nextNum+1].row === 0) ){
            moveRowLeft(rows-1, position, res)
            moveColUp(cols-1, position,res)
          } else {
            while(bottomRow[cols-1] !== nextNum+1){
              moveRowRight(rows-1, position, res)
              moveColUp(cols-1, position, res)
            }
          }
        }
        break
      }
    }
    // Dealing with the parity issues
    if((isAsc(position, solvedBoard) && 
      position[solvedBoard[0][cols-1]].row === 0 && position[solvedBoard[0][cols-1]].col === cols-1) || ( isAsc(position, solvedBoard) && position[solvedBoard[rows-2][cols-1]].row === 0 && position[solvedBoard[rows-2][cols-1]].col === cols-1)){
      if(cols === 2){
        if(position[solvedBoard[rows-2][cols-1]].row === rows-1){
          if(position[solvedBoard[rows-2][cols-1]].col === cols-1){
            moveRowLeft(rows-1, position, res)
          }
          moveColDown(cols-1, position, res)
          moveRowRight(rows-1, position, res)
          moveColUp(cols-1, position, res)
        } else{
        moveRowLeft(rows-1, position, res)
        }
        break
      } else if ((rows > 3 || cols > 4) && rowEven){
        // If the rows are even
        while(position[solvedBoard[rows-1][0]].col !== cols-1){
          moveRowRight(rows-1, position, res)
        }
        moveRowLeft(rows-1, position, res)
        moveColUp(cols-1, position, res)
        moveRowRight(rows-1, position, res)
        moveColUp(cols-1, position, res)
        rowEven = !rowEven
      } else if((rows > 3 || cols > 4) && !rowEven){
        moveColUp(cols-1, position, res)
        moveRowLeft(rows-1, position, res)
        moveColDown(cols-1, position, res)
        moveRowLeft(rows-1, position, res)
        rowEven = !rowEven
        // if the rows are odd
      } else {
        // If the grid is 3*3 or 3*2
        if(isAsc(position, solvedBoard)) break
        moveRowLeft(rows-1, position, res)
        moveColDown(cols-1, position, res)
        moveRowLeft(rows-1, position, res)
        moveColUp(cols-1, position, res)
        moveRowLeft(rows-1, position, res)
      }
      colNumInOrder(cols, rows, position, solvedBoard,res)
    }
  }
  // shifting the last row until it matches with the solution
  let lastRow = getBottomRow(position, solvedBoard)
  if(lastRow.includes(solvedBoard[solvedBoard.length-2][solvedBoard[0].length-1])){
    while(solvedBoard[solvedBoard.length-2][solvedBoard[0].length-1] !== lastRow[lastRow.length-1]){
      moveRowRight(solvedBoard.length-1, position, res)
      let lastElem = lastRow.pop()
      lastRow.unshift(lastElem)
    }
    moveColUp(solvedBoard[0].length-1, position, res)
  }
  while(!isMatching(position, solvedBoard)){
    moveRowLeft(solvedBoard.length-1, position, res)
  }
  return res
}
// Check if the solution is reachable
function isMatching(position, solvedBoard){
  for(let i = 0 ; i < solvedBoard.length ; i++){
    for(let j = 0 ; j < solvedBoard[0].length ; j++){
      if(position[solvedBoard[i][j]].row !== i || position[solvedBoard[i][j]].col !== j){
        return false
      }
    }
  }
  return true
}

function colNumInOrder(cols, rows, position, solvedBoard, res){
  if(rows === 2) return
  for(let i = 0 ; i < rows-2 ; i++){
    if(position[solvedBoard[i][cols-1]].row === rows-1){
      while(position[solvedBoard[i][cols-1]].col !== cols-1){
        moveRowRight(rows-1, position, res)
      }
      moveColUp(cols-1, position, res)
    } else {
      if(i === 0){
        while(position[solvedBoard[i][cols-1]].row !== rows-2){
          moveColDown(cols-1, position, res)
        }
      } else {
        while(position[solvedBoard[i][cols-1]].row !== rows-1){
        moveColDown(cols-1, position, res)
      }
      moveRowRight(rows-1, position, res)
      while(position[solvedBoard[i-1][cols-1]].row !== rows-2){
        moveColDown(cols-1, position, res)
      }
      moveRowLeft(rows-1, position, res)
      moveColUp(cols-1, position, res)
      }
    }
  }

  while(position[solvedBoard[0][cols-1]].row !== 0){
    moveColUp(cols-1, position, res)
  }

  if(position[solvedBoard[rows-1][0]].col === cols-1 && position[solvedBoard[rows-1][0]].row === rows-2){
    moveColDown(cols-1, position, res)
    moveRowLeft(rows-1, position, res)
    moveColUp(cols-1, position, res)
  }
}

function numToExchange(row, col, position){
  let num = 0
  for(const key in position){
    if(position[key].row === row && position[key].col === col){
      num = Number(key)
      break
    }
  }
  return num
}
// Check if the last row is in ascending order
function isAsc(position, solvedBoard){
  let bottomRow = getBottomRow(position, solvedBoard)
  let target = solvedBoard[solvedBoard.length-2][solvedBoard[0].length-1]
  let dontTouch = bottomRow.indexOf(target)
  if(dontTouch !== -1){
    bottomRow[dontTouch] = bottomRow[dontTouch-1]+1 || bottomRow[bottomRow.length-1]+1
  }
  let start = bottomRow.indexOf(solvedBoard[solvedBoard.length-1][0])
  for(let i = start+1 ; i < start + bottomRow.length ; i++){
    if(bottomRow[i % bottomRow.length] !== bottomRow[(i-1) % bottomRow.length] + 1){
      return false
    }
  }
  return true
}

function getBottomRow(position, solvedBoard){
    let bottomRow = []
  for(const key in position){
    if(position[key].row === solvedBoard.length-1){
      bottomRow.push({key : Number(key), col : position[key].col})
    }
  }
  return bottomRow = bottomRow.sort((a, b) => a.col - b.col).map(x => x.key)
}

function moveRowRight(row, position, res){
  let arrToMove = []
  for(const key in position){
    if(position[key].row === row){
      arrToMove.push({key : Number(key), col : position[key].col})
    }
  }
  arrToMove.sort((a, b) => a.col - b.col) 
  let lastNum = arrToMove.pop()
  arrToMove.unshift(lastNum)
  arrToMove.forEach((elem, ind) => position[elem.key].col = ind)
  res.push(`R${row}`)
}

function moveRowLeft(row, position, res){
  let rowElements = []
  for(const key in position){
    if(position[key].row === row){
      rowElements.push({ key : Number(key), col : position[key].col })
    }
  }
  rowElements.sort((a, b) => a.col - b.col)
  let firstElem = rowElements.shift()
  rowElements.push(firstElem)
  rowElements.forEach((x,i) => position[x.key].col = i)
  res.push(`L${row}`)
}

function moveColUp(col, position, res){
  let colElements = []
  for(const key in position){
    if(position[key].col === col){
      colElements.push({key : Number(key), row : position[key].row})
    }
  }
  colElements.sort((a,b) => a.row - b.row)
  let firstNum = colElements.shift()
  colElements.push(firstNum)
  colElements.forEach((x,i) => position[x.key].row = i)
  res.push(`U${col}`)
}

function moveColDown(col, position, res){
  let colElements = []
  for(const key in position){
    if(position[key].col === col){
      colElements.push({key : Number(key), row : position[key].row})
    }
  }
  colElements.sort((a, b) => a.row - b.row)
  let lastElement = colElements.pop()
  colElements.unshift(lastElement)
  colElements.forEach((elem, i) => position[elem.key].row = i)
  res.push(`D${col}`)
}

function isSolvable(board, solvedBoard) {
  const rows = solvedBoard.length
  const cols = solvedBoard[0].length
  const letterToNum = {}
  solvedBoard.flat().forEach((elem, i) => letterToNum[elem] = i + 1)
  const flat = board.flat().map(x => letterToNum[x])
  if (flat.some(x => x === undefined)) return false
  if ((rows % 2 === 1) && (cols % 2 === 1)) {
    let inversions = 0
    for (let i = 0; i < flat.length; i++) {
      for (let j = i + 1; j < flat.length; j++) {
        if (flat[i] > flat[j]) inversions++
      }
    }
    return inversions % 2 === 0
  }
  return true
}

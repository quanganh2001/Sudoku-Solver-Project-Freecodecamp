# Install package and config environment
## Install package
Type: `npm i`
## Config environment
Create new project then create cluster yourself.
Setup username and password cluster to environment file:
```env
PORT=3000
NODE_ENV=test
```
# Sudoku solver
```js
class SudokuSolver {
  validate(puzzleString) {}

  letterToNumber(row) {
    switch (row.toUpperCase()) {
      case "A":
        return 1;
      case "B":
        return 2;
      case "C":
        return 3;
      case "D":
        return 4;
      case "E":
        return 5;
      case "F":
        return 6;
      case "G":
        return 7;
      case "H":
        return 8;
      case "I":
        return 9;
      default:
        return "none";
    }
  }

  checkRowPlacement(puzzleString, row, column, value) {
    let grid = this.transform(puzzleString);
    row = this.letterToNumber(row);
    
    if (grid[row - 1][column - 1] == value){
      return true;
    } 
    for (let i = 0; i < 9; i++) {
      if (grid[row - 1][i] == value) {
        return false;
      }
    }
    return true;
  }

  checkColPlacement(puzzleString, row, column, value) {
    let grid = this.transform(puzzleString);
    row = this.letterToNumber(row);
    
    if (grid[row - 1][column - 1] == value){
      return true;
    } 
    for (let i = 0; i < 9; i++) {
      if (grid[i][column - 1] == value) {
        return false;
      }
    }
    return true;
  }

  checkRegionPlacement(puzzleString, row, col, value) {
    let grid = this.transform(puzzleString);
    row = this.letterToNumber(row);
    
    if (grid[row - 1][col - 1] == value){
      return true;
    } 
    let startRow = row - (row % 3),
      startCol = col - (col % 3);
    for (let i = 0; i < 3; i++)
      for (let j = 0; j < 3; j++)
        if (grid[i + startRow][j + startCol] == value) return false;
    return true;
  }

  solveSuduko(grid, row, col) {
    const N = 9;

    if (row == N - 1 && col == N) return grid;

    if (col == N) {
      row++;
      col = 0;
    }

    if (grid[row][col] != 0) return this.solveSuduko(grid, row, col + 1);

    for (let num = 1; num < 10; num++) {
      if (this.isSafe(grid, row, col, num)) {
        grid[row][col] = num;

        if (this.solveSuduko(grid, row, col + 1)) return grid;
      }

      grid[row][col] = 0;
    }
    return false;
  }

  isSafe(grid, row, col, num) {
    // Check if we find the same num
    // in the similar row , we
    // return false
    for (let x = 0; x <= 8; x++) if (grid[row][x] == num) return false;

    // Check if we find the same num
    // in the similar column ,
    // we return false
    for (let x = 0; x <= 8; x++) if (grid[x][col] == num) return false;

    // Check if we find the same num
    // in the particular 3*3
    // matrix, we return false
    let startRow = row - (row % 3),
      startCol = col - (col % 3);
    for (let i = 0; i < 3; i++)
      for (let j = 0; j < 3; j++)
        if (grid[i + startRow][j + startCol] == num) return false;

    return true;
  }

  transform(puzzleString) {
    // take ..53..23.23. => [[0,0,5,3,0,0,2,3,0],
    // [2,3,0]
    let grid = [
      [0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 0, 0, 0],
    ];
    let row = -1;
    let col = 0;
    for (let i = 0; i < puzzleString.length; i++) {
      if (i % 9 == 0) {
        row++;
      }
      if (col % 9 == 0) {
        col = 0;
      }

      grid[row][col] = puzzleString[i] === "." ? 0 : +puzzleString[i];
      col++;
    }
    return grid;
  }

  transformBack(grid) {
    return grid.flat().join("");
  }

  solve(puzzleString) {
    if (puzzleString.length != 81) {
      return false;
    }
    if (/[^0-9.]/g.test(puzzleString)) {
      return false;
    }
    let grid = this.transform(puzzleString);
    let solved = this.solveSuduko(grid, 0, 0);
    if (!solved) {
      return false;
    }
    let solvedString = this.transformBack(solved);
    console.log("solvedString :>> ", solvedString);
    return solvedString;
  }
}

module.exports = SudokuSolver;
```
# API Routes
```js
"use strict";

const SudokuSolver = require("../controllers/sudoku-solver.js");

module.exports = function(app) {
  let solver = new SudokuSolver();

  app.route("/api/check").post((req, res) => {
    const { puzzle, coordinate, value } = req.body;
    if (!puzzle || !coordinate || !value) {
      res.json({ error: "Required field(s) missing" });
      return;
    }
    const row = coordinate.split("")[0];
    const column = coordinate.split("")[1];
    if (
      coordinate.length !== 2 ||
      !/[a-i]/i.test(row) ||
      !/[1-9]/i.test(column)
    ) {

      res.json({ error: "Invalid coordinate" });
      return;
    }
    if (/[^1-9]/ig.test(value)) {
      res.json({ error: "Invalid value" });
      return;
    }
    if (puzzle.length != 81) {
      res.json({ error: "Expected puzzle to be 81 characters long" });
      return;
    }
    if (/[^1-9.]/g.test(puzzle)) {
      res.json({ error: "Invalid characters in puzzle" });
      return;
    }
    let validCol = solver.checkColPlacement(puzzle, row, column, value);
    let validReg = solver.checkRegionPlacement(puzzle, row, column, value);
    let validRow = solver.checkRowPlacement(puzzle, row, column, value);
    let conflicts = [];

    if (validCol && validReg && validRow) {
      res.json({ valid: true });
      return;
    } else {
      if (!validRow) {
        conflicts.push("row");
      }
      if (!validCol) {
        conflicts.push("column");
      }
      if (!validReg) {
        conflicts.push("region");
      }
      res.json({ valid: false, conflict: conflicts });
      return;
    }
  });

  app.route("/api/solve").post((req, res) => {
    const { puzzle } = req.body;
    if (!puzzle) {
      res.json({ error: "Required field missing" });
      return;
    }
    if (puzzle.length != 81) {
      res.json({ error: "Expected puzzle to be 81 characters long" });
      return;
    }
    if (/[^0-9.]/g.test(puzzle)) {
      res.json({ error: "Invalid characters in puzzle" });
      return;
    }
    let solvedString = solver.solve(puzzle);
    if (!solvedString) {
      res.json({ error: "Puzzle cannot be solved" });
    } else {
      res.json({ solution: solvedString });
    }
  });
};
```
# Tests
## Unit tests
```js
const chai = require("chai");
const assert = chai.assert;

const Solver = require("../controllers/sudoku-solver.js");
let solver = new Solver();

let validPuzzle =
  "1.5..2.84..63.12.7.2..5.....9..1....8.2.3674.3.7.2..9.47...8..1..16....926914.37.";
suite("UnitTests", () => {
  suite("solver tests", function () {
    test("Logic handles a valid puzzle string of 81 characters", function (done) {
      let complete =
        "135762984946381257728459613694517832812936745357824196473298561581673429269145378";
      assert.equal(solver.solve(validPuzzle), complete);
      done();
    });

    test("Logic handles a puzzle string with invalid characters (not 1-9 or .)", function (done) {
      let inValidPuzzle =
        "1.5..2.84..63.12.7.2..5..g..9..1....8.2.3674.3.7.2..9.47...8..1..16....926914.37.";
      assert.equal(solver.solve(inValidPuzzle), false);
      done();
    });

    test("Logic handles a puzzle string that is not 81 characters in length", function (done) {
      let inValidPuzzle =
        "1.5..2.84..63.12.7.2..5.......9..1....8.2.3674.3.7.2..9.47...8..1..16....926914.37.";
      assert.equal(solver.solve(inValidPuzzle), false);
      done();
    });

    test("Logic handles a valid row placement", function (done) {
      assert.equal(solver.checkRowPlacement(validPuzzle, "A", "2", "9"), true);
      done();
    });

    test("Logic handles an invalid row placement", function (done) {
      assert.equal(solver.checkRowPlacement(validPuzzle, "A", "2", "1"), false);
      done();
    });

    test("Logic handles a valid column placement", function (done) {
      assert.equal(solver.checkColPlacement(validPuzzle, "A", "2", "8"), true);
      done();
    });

    test("Logic handles an invalid column placement", function (done) {
      assert.equal(solver.checkColPlacement(validPuzzle, "A", "2", "9"), false);
      done();
    });

    test("Logic handles a valid region (3x3 grid) placement", function (done) {
      assert.equal(
        solver.checkRegionPlacement(validPuzzle, "A", "2", "3"),
        true
      );
      done();
    });

    test("Logic handles an invalid region (3x3 grid) placement", function (done) {
      assert.equal(
        solver.checkRegionPlacement(validPuzzle, "A", "2", "1"),
        false
      );
      done();
    });
    test("Valid puzzle strings pass the solver", function (done) {
      assert.equal(
        solver.solve(validPuzzle),
        "135762984946381257728459613694517832812936745357824196473298561581673429269145378"
      );
      done();
    });
    test("Invalid puzzle strings fail the solver", function (done) {
      let inValidPuzzle =
        "115..2.84..63.12.7.2..5.....9..1....8.2.3674.3.7.2..9.47...8..1..16....926914.37.";
      assert.equal(solver.solve(inValidPuzzle), false);
      done();
    });
    test("Solver returns the the expected solution for an incomplete puzzzle", function (done) {
      assert.equal(
        solver.solve(
          "..839.7.575.....964..1.......16.29846.9.312.7..754.....62..5.78.8...3.2...492...1"
        ),
        "218396745753284196496157832531672984649831257827549613962415378185763429374928561"
      );
      done();
    });
  });
});
```
## Functional Tests
```js
const chai = require("chai");
const chaiHttp = require("chai-http");
const assert = chai.assert;
const server = require("../server");

chai.use(chaiHttp);

// Solve a puzzle with valid puzzle string: POST request to /api/solve
// Solve a puzzle with missing puzzle string: POST request to /api/solve
// Solve a puzzle with invalid characters: POST request to /api/solve
// Solve a puzzle with incorrect length: POST request to /api/solve
// Solve a puzzle that cannot be solved: POST request to /api/solve
// Check a puzzle placement with all fields: POST request to /api/check

let validPuzzle =
  "1.5..2.84..63.12.7.2..5.....9..1....8.2.3674.3.7.2..9.47...8..1..16....926914.37.";
suite("Functional Tests", () => {
  test("Solve a puzzle with valid puzzle string: POST request to /api/solve", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/solve")
      .send({ puzzle: validPuzzle })
      .end(function (err, res) {
        assert.equal(res.status, 200);
        let complete =
          "135762984946381257728459613694517832812936745357824196473298561581673429269145378";
        assert.equal(res.body.solution, complete);
        done();
      });
  });

  test("Solve a puzzle with missing puzzle string: POST request to /api/solve", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/solve")
      .send({})
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(res.body.error, "Required field missing");
        done();
      });
  });

  test("Solve a puzzle with invalid characters: POST request to /api/solve", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/solve")
      .send({
        puzzle:
          "1.5..2.84..63.12.7.2..5..h..9..1....8.2.3674.3.7.2..9.47...8..1..16....926914.37.",
      })
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(res.body.error, "Invalid characters in puzzle");
        done();
      });
  });

  test("Solve a puzzle with incorrect length: POST request to /api/solve", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/solve")
      .send({
        puzzle:
          "1.5..2.84..63.12.7.2..5...9..1....8.2.3674.3.7.2..9.47...8..1..16....926914.37.",
      })
    
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(
          res.body.error,
          "Expected puzzle to be 81 characters long"
        );
        done();
      });
  });

  test("Solve a puzzle that cannot be solved: POST request to /api/solve", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/solve")
      .send({
        puzzle:
          "115..2.84..63.12.7.2..5.....9..1....8.2.3674.3.7.2..9.47...8..1..16....926914.37.",
      })
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(res.body.error, "Puzzle cannot be solved");
        done();
      });
  });
  test("Check a puzzle placement with all fields: POST request to /api/check", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/check")
      .send({ puzzle: validPuzzle, coordinate: "A2", value: "3" })
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(res.body.valid, true);
        done();
      });
  });

  test("Check a puzzle placement with single placement conflict: POST request to /api/check", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/check")
      .send({ puzzle: validPuzzle, coordinate: "A2", value: "8" })
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(res.body.valid, false);
        assert.equal(res.body.conflict.length, 1);
        done();
      });
  });
  test("Check a puzzle placement with multiple placement conflicts: POST request to /api/check", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/check")
      .send({ puzzle: validPuzzle, coordinate: "A2", value: "6" })
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(res.body.valid, false);
        assert.equal(res.body.conflict.length, 2);
        done();
      });
  });
  test("Check a puzzle placement with all placement conflicts: POST request to /api/check", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/check")
      .send({ puzzle: validPuzzle, coordinate: "A2", value: "2" })
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(res.body.valid, false);
        assert.equal(res.body.conflict.length, 3);
        done();
      });
  });
  test("Check a puzzle placement with missing required fields: POST request to /api/check", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/check")
      .send({ puzzle: validPuzzle, value: "3" })
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(res.body.error, "Required field(s) missing");
        done();
      });
  });
  test("Check a puzzle placement with invalid characters: POST request to /api/check", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/check")
      .send({
        puzzle:
          "1.5..2.84..63.12.7.2..5..h..9..1....8.2.3674.3.7.2..9.47...8..1..16....926914.37.",
        coordinate: "A2",
        value: "3",
      })
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(res.body.error, "Invalid characters in puzzle");
        done();
      });
  });
  test("Check a puzzle placement with incorrect length: POST request to /api/check", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/check")
      .send({
        puzzle:
          "1.5..2.84..63.12.7.2..5..h..9..1..8.2.3674.3.7.2..9.47...8..1..16....926914.37.",
        coordinate: "A2",
        value: "3",
      })
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(
          res.body.error,
          "Expected puzzle to be 81 characters long"
        );
        done();
      });
  });
  test("Check a puzzle placement with invalid placement coordinate: POST request to /api/check", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/check")
      .send({ puzzle: validPuzzle, coordinate: "L2", value: "3" })
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(res.body.error, "Invalid coordinate");
        done();
      });
  });
  test("Check a puzzle placement with invalid placement value: POST request to /api/check", function (done) {
    chai
      .request(server)
      .keepOpen()
      .post("/api/check")
      .send({ puzzle: validPuzzle, coordinate: "A2", value: "g" })
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.equal(res.body.error, "Invalid value");
        done();
      });
  });
});
```
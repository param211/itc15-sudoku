<script>
  import * as jQuery from "jquery";
  import "bootstrap/dist/css/bootstrap.min.css";
  import "bootstrap/dist/js/bootstrap.bundle.min.js";
  import confetti from "canvas-confetti";
  import { debounce } from "debounce";

  let blankFields = 64;
  let filledFields = 0;
  let allDefined = false;
  let allValid = false;
  let generatedGrid;

  const shuffle = (arr, s = 0, e = arr.length - 1, axis = 0) => {
    for (let i = e; i > s; i--) {
      let j = Math.floor(Math.random() * (i - s + 1) + s);
      if (axis == 0) [arr[i], arr[j]] = [arr[j], arr[i]];
      else {
        for (let k = 0; k < arr.length; k++) {
          [arr[k][i], arr[k][j]] = [arr[k][j], arr[k][i]];
        }
      }
    }
  };

  const generateRandomRow = () => {
    let range = [...new Array(9)].map((_, i) => i + 1);
    let row = [...new Array(9)];
    for (const i in row)
      row[i] = range.splice(Math.floor(Math.random() * range.length), 1)[0];
    return row;
  };

  const createSudokuBoard = blanks => {
    generatedGrid = [];
    generatedGrid.push(generateRandomRow());
    for (let i = 1; i < 9; i++) {
      let n = !(i % 3) ? 1 : 3;
      generatedGrid.push([
        ...generatedGrid[i - 1].slice(n),
        ...generatedGrid[i - 1].slice(0, n)
      ]);
    }

    generatedGrid.forEach(row => {
      for (let i in row) {
        row[i] = { value: row[i], editable: false, valid: true };
      }
    });

    // Should probably shuffle the 3x3 grids instead of rows so you can't just recognize the pattern...
    shuffle(generatedGrid, 0, 2, 0);
    shuffle(generatedGrid, 3, 5, 0);
    shuffle(generatedGrid, 6, 8, 0);
    shuffle(generatedGrid, 0, 2, 1);
    shuffle(generatedGrid, 3, 5, 1);
    shuffle(generatedGrid, 6, 8, 1);

    for (let n = 81; n > 81 - blanks; ) {
      let x = Math.floor(Math.random() * 9);
      let y = Math.floor(Math.random() * 9);
      if (generatedGrid[y][x].value) {
        generatedGrid[y][x] = { value: undefined, editable: true, valid: true };
        n--;
      }
    }
  };

  //createSudokuBoard();

  $: createSudokuBoard(blankFields);

  const validateBoard_debounced = debounce(validateBoard, 100);

  function validatePosition(y, x) {
    return function() {
      let c = generatedGrid[y][x];
      let v = true;
      for (let i = 0; i < 9; i++) {
        if (c.value && i != x && c.value == generatedGrid[y][i].value) {
          v = false;
        }
        if (c.value && i != y && c.value == generatedGrid[i][x].value) {
          v = false;
        }
        if (
          c.value &&
          3 * ((y / 3) | 0) + ((i / 3) | 0) != y &&
          3 * ((x / 3) | 0) + (i % 3) != x &&
          c.value ==
            generatedGrid[3 * ((y / 3) | 0) + ((i / 3) | 0)][
              3 * ((x / 3) | 0) + (i % 3)
            ].value
        ) {
          v = false;
        }
      }
      return (generatedGrid[y][x].valid = v);
    };
  }

  function validateBoard() {
    allDefined = true;
    allValid = true;
    for (let i = 0; i < 9; i++) {
      for (let j = 0; j < 9; j++) {
        if (!generatedGrid[i][j].value) {
          allDefined = false;
        }
        if (!validatePosition(i, j)()) {
          allValid = false;
        }
      }
    }
    allDefined &&
      allValid &&
      jQuery("#modalWindow").modal({ backdrop: false, show: true }) &&
      confetti({
        particleCount: 100,
        angle: 0,
        spread: 100,
        origin: { y: 0.4 }
      }) &&
      confetti({
        particleCount: 100,
        angle: 180,
        spread: 100,
        origin: { y: 0.4 }
      });
  }
</script>

<style>
  input[type="number"]::-webkit-outer-spin-button,
  input[type="number"]::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }
  input[type="number"] {
    -moz-appearance: textfield;
  }
  .cell {
    display: inline-block;
    font-size: 24px;
    line-height: 50px;
    width: 50px;
    height: 50px;
    text-align: center;
    vertical-align: text-bottom;
    box-sizing: border-box;
  }
  .board {
    width: 530px;
  }
  @media (max-width: 720px) {
    .cell {
      font-size: 14px;
      line-height: 24px;
      height: 24px;
      width: 24px;
    }
    .board {
      width: 290px;
    }
  }
</style>

<div class="container">
  <div
    class="modal fade"
    id="modalWindow"
    tabindex="-1"
    role="dialog"
    aria-labelledby="modalWindowLabel"
    aria-hidden="true">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="modalWindowLabel">You solved it!</h5>
          <button
            type="button"
            class="close"
            data-dismiss="modal"
            aria-label="Close">
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          Unfortunately, the only prize is the confetti.
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-primary" data-dismiss="modal">
            Close
          </button>
        </div>
      </div>
    </div>
  </div>
  <div class="board mx-auto">
    <label for="difficultySlider" class="mb-n1">Difficulty Level:</label>
    <input
      type="range"
      style="font-size: 8px"
      class="custom-range"
      min="0"
      max="64"
      step="1"
      bind:value={blankFields}
      id="difficultySlider"
      name="difficultySlider" />
    {#each generatedGrid as row, i}
      <div
        class={[3, 6].includes(i) ? 'border-top border-dark' : ''}
        style="text-align: center;">
        {#each Array(3) as _, j}
          <div
            style="display: inline-block; box-sizing: border-box;"
            class={j == 1 || j == 2 ? 'border-left border-dark' : ''}>
            {#each Array(3) as _, k}
              {#if row[j * 3 + k].editable}
                <input
                  type="number"
                  class="cell border rounded bg-light text-secondary m-1"
                  class:border-danger={!row[j * 3 + k].valid}
                  min="1"
                  step="1"
                  max="9"
                  bind:value={row[j * 3 + k].value}
                  on:change={validateBoard_debounced} />
              {:else}
                <div
                  class="cell border rounded bg-dark text-light m-1"
                  class:border-danger={!row[j * 3 + k].valid}>
                  {row[j * 3 + k].value}
                </div>
              {/if}
            {/each}
          </div>
        {/each}
      </div>
    {/each}
  </div>
</div>

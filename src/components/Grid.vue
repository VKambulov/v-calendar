<script>
const directions = {
  vLeading: 'vertical-leading',
  vTrailing: 'vertical-trailing',
  hLeading: 'horizontal-leading',
  hTrailing: 'horizontal-trailing',
};
export default {
  name: 'Grid',
  render(h) {
    // Grid cell renderer
    const getCell = ({ nodes, position, row, column }) => {
      // Get the default slot first
      if (nodes.length >= position) {
        return nodes[position - 1];
      }
      // Get the scoped slot second
      if (this.$scopedSlots.default) {
        return this.$scopedSlots.default({
          position,
          row,
          column,
        });
      }
      return null;
    };

    // Grid cells renderer
    const getCells = () => {
      const rows = [];
      // Resolve default slot nodes (remove whitespaced)
      const nodes =
        (this.$slots.default &&
          this.$slots.default.filter(n => n.tag !== undefined)) ||
        [];
      // Build cells
      for (let r = 1, p = 1; r <= this.rows; r++) {
        const rowCells = [];

        for (let c = 1; c <= this.columns; c++) {
          // Add the cell for current row & column
          rowCells.push(
            h(
              'th',
              {
                style: {
                  'padding': 0
                },
                on: {
                  keydown: e =>
                    this.handleCellKeydown({ row: r, column: c, event: e }),
                },
              },
              [getCell({ nodes, position: p++, row: r, column: c })],
            ),
          );
        }

        rows.push(
          h(
            'tr',
            {},
            [
              rowCells
            ],
          ),
        );
      }
      return rows;
    };

    return h(
      'table',
      {
        //class: 'vc-grid-container',
        style: this.containerStyle,
      },
      [...getCells()],
    );
  },
  props: {
    count: Number,
    rows: {
      type: Number,
      default: 1,
    },
    columns: {
      type: Number,
      default: 1,
    },
    gap: {
      type: String,
      default: '0px',
    },
    autofit: Boolean,
    columnWidth: {
      type: String,
      default: '1fr',
    },
    disableFocus: {
      type: Boolean,
      default: false,
    },
  },
  computed: {
    containerStyle() {
      return {
        width: '100%',
        display: 'inline-table',
        'text-align': 'center',
        'border-collapse': 'unset !important'
      };
    }
  },
  methods: {
    handleCellKeydown({ row, column, event }) {
      // Return if focus management is disabled
      if (this.disableFocus) return;
      const state = {
        row,
        column,
        alt: false,
        handled: false,
      };
      // Increment row/column based on key
      switch (event.key) {
        case 'ArrowUp': {
          state.row--;
          break;
        }
        case 'ArrowDown': {
          state.row++;
          break;
        }
        case 'ArrowLeft': {
          state.column--;
          break;
        }
        case 'ArrowRight': {
          state.column++;
          break;
        }
        case 'Home': {
          state.column = 1;
          break;
        }
        case 'End': {
          state.column = this.columns;
          break;
        }
        case 'PageUp': {
          state.alt = event.altKey;
          state.direction = directions.vLeading;
          break;
        }
        case 'PageDown': {
          state.alt = event.altKey;
          state.direction = directions.vTrailing;
          break;
        }
        default: {
          return;
        }
      }
      // Handle state for row rollovers
      if (state.row < 1) {
        state.direction = directions.vLeading;
        state.row = this.rows;
      } else if (state.row > this.rows) {
        state.direction = directions.vTrailing;
        state.row = 1;
      }
      // Handle state for column rollovers
      if (state.column < 1) {
        state.direction = directions.hLeading;
        state.column = this.columns;
      } else if (state.column > this.columns) {
        state.direction = directions.hTrailing;
        state.column = 1;
      }
      // Emit rollover event if direction was assigned
      if (state.direction) {
        this.$emit('rollover', state);
      }
      // Focusd on cell for current state if event wasn't handled
      if (!state.handled) {
        // Get grid cell element
        const cellSelector = `.vc-grid-cell-row-${state.row}.vc-grid-cell-col-${state.column}`;
        const cellEl = this.$el.querySelector(cellSelector);
        if (cellEl) {
          this.tryFocus(cellEl);
        }
      }
      event.stopPropagation();
      event.preventDefault();
    },
    tryFocus(el = this.$el) {
      this.$nextTick(() => {
        const selectors = [
          '.vc-grid-focus',
          'button, [href], input, select, textarea, [tabindex="0"]',
          '[tabindex]:not([tabindex="undefined"])',
        ];
        const focusableEl = selectors
          .map(s => el.querySelector(s))
          .find(e => e);
        if (focusableEl) {
          focusableEl.focus();
          return true;
        }
        return false;
      });
    },
  },
};
</script>

<style scoped>
.vc-grid-container {
  position: relative;
  flex-shrink: 1;
  display: grid;
  overflow: auto;
  -webkit-overflow-scrolling: touch;
}
.vc-grid-cell {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>

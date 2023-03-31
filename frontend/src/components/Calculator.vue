<template>
  <div class="calculator">
    <input type="text" class="calculator-input" v-model="expression" readonly>
    <div class="calculator-buttons">
      <CalcButton v-for="button in buttons" :key="button.label" :label="button.label"
        @click="handleButtonClick(button.action)"></CalcButton>
      <CalcButton label="History" @click="handleButtonClick('History')" class="double-width"></CalcButton>
      <CalcButton label="Clear History" @click="handleButtonClick('Clear History')" class="double-width"></CalcButton>
    </div>
    <div v-if="showHistoryModal" class="modal is-active">
      <div class="modal-background" @click="showHistoryModal = false"></div>
      <div class="modal-card">
        <header class="modal-card-head">
          <p class="modal-card-title">Calculation History</p>
        </header>
        <section class="modal-card-body">
          <table class="table is-striped is-hoverable is-fullwidth">
            <thead>
              <tr>
                <th>Exp</th>
                <th>Result</th>
                <th>Delete</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(record, index) in historyRecords" :key="index">
                <td>{{ record.expression }}</td>
                <td>{{ record.result }}</td>
                <td>
                  <i class="fas fa-trash-alt delete-history" @click="deleteHistoryRecord(record.id)"></i>
                </td>
              </tr>
            </tbody>
          </table>
        </section>

        <footer class="modal-card-foot">
          <button class="button" @click="showHistoryModal = false">Close</button>
        </footer>
      </div>
    </div>
  </div>
</template>
  
<script>
import CalcButton from '@/components/CalcButton.vue';
import calculationHistoryService from '@/services/calculationHistoryService';
import * as math from 'mathjs';

export default {
  name: 'CalculatorButtons',
  components: {
    CalcButton,
  },
  data() {
    return {
      expression: '',
      showHistoryModal: false,
      historyRecords: [],
      buttons: [
        { label: '7', action: '7' },
        { label: '8', action: '8' },
        { label: '9', action: '9' },
        { label: '+', action: '+' },
        { label: '4', action: '4' },
        { label: '5', action: '5' },
        { label: '6', action: '6' },
        { label: '-', action: '-' },
        { label: '1', action: '1' },
        { label: '2', action: '2' },
        { label: '3', action: '3' },
        { label: '*', action: '*' },
        { label: '0', action: '0' },
        { label: 'C', action: 'C' },
        { label: '=', action: '=' },
        { label: '/', action: '/' },
      ],
    };
  },
  methods: {
    async fetchHistory() {
      this.historyRecords = await calculationHistoryService.getAllHistory();
    },
    async deleteHistoryRecord(id) {
      await calculationHistoryService.deleteHistoryById(id);
      await this.fetchHistory();
    },
    async deleteAllHistoryRecords() {
      await calculationHistoryService.deleteAllHistory();
      await this.fetchHistory();
    },
    async handleEquals() {
      try {
        const result = this.evaluateExpression(this.expression);
        if (result !== null) {
          const history = {
            expression: this.expression,
            result: result,
          };
          this.expression = result.toString();

          await calculationHistoryService.createHistory(history);
        } else {
          console.error('Invalid expression');
        }
      } catch (error) {
        console.error('Error performing calculation:', error);
      }
    },
    handleClear() {
      this.expression = '';
    },
    async handleHistory() {
      this.showHistoryModal = true;
      await this.fetchHistory();
    },
    async handleClearHistory() {
      await this.deleteAllHistoryRecords();
    },
    isOperator(char) {
      return ['+', '-', '*', '/'].includes(char);
    },
    isLastCharOperator() {
      const lastChar = this.expression.slice(-1);
      return this.isOperator(lastChar);
    },
    handleInput(value) {
      if (this.isOperator(value)) {
        const isDisallowedFirstChar = this.expression.length === 0 && (value === '*' || value === '/');
        if (isDisallowedFirstChar || (this.isLastCharOperator() && (this.expression.length === 1 && (value === '*' || value === '/')))) {
          return;
        }

        if (this.isLastCharOperator()) {
          this.expression = this.expression.slice(0, -1);
        }
      }
      this.expression += value;
    },
    async handleButtonClick(value) {
      switch (value) {
        case '=':
          await this.handleEquals();
          break;
        case 'C':
          this.handleClear();
          break;
        case 'History':
          await this.handleHistory();
          break;
        case 'Clear History':
          await this.handleClearHistory();
          break;
        default:
          this.handleInput(value);
          break;
      }
    },
    evaluateExpression(expression) {
      try {
        return math.evaluate(expression);
      } catch (error) {
        console.error('Error evaluating expression:', error);
        return null;
      }
    },
  },
};
</script>
<style scoped>
.calculator {
  max-width: 300px;
  margin: 0 auto;
  text-align: center;
}

.calculator-input {
  padding: 10px;
  margin-bottom: 10px;
  font-size: 24px;
  text-align: right;
  border: none;
  background-color: #f2f2f2;
}

.calculator-buttons {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-gap: 10px;
}

.double-width {
  grid-column-end: span 2;
}

.modal {
  align-items: center;
  display: flex;
  height: 100%;
  justify-content: center;
  left: 0;
  position: fixed;
  top: 0;
  width: 100%;
  z-index: 40;
}

.modal.is-active {
  z-index: 2000;
}

.modal-background {
  background-color: rgba(10, 10, 10, 0.86);
  height: 100%;
  left: 0;
  position: absolute;
  top: 0;
  width: 100%;
}

.modal-card {
  background-color: #fff;
  border-radius: 6px;
  box-shadow: 0 10px 20px rgba(10, 10, 10, 0.19), 0 6px 6px rgba(10, 10, 10, 0.23);
  max-width: 500px;
  position: relative;
  width: auto;
}

.modal-card-head,
.modal-card-foot {
  align-items: center;
  background-color: #f5f5f5;
  display: flex;
  flex-shrink: 0;
  justify-content: flex-start;
  padding: 20px;
}

.modal-card-title {
  flex-grow: 1;
  flex-shrink: 0;
  font-size: 1.5rem;
  line-height: 1;
}

.modal-card-body {
  overflow-y: auto;
  padding: 20px;
  max-height: 400px;
}

.table {
  font-size: 0.9rem;
  margin: auto;
}
.table thead {
  background-color: #f5f5f5;
}

.table tbody tr:hover {
  background-color: #f5f5f5;
}
.delete-history {
  cursor: pointer;
  margin-left: 8px;
  color: rgb(179, 179, 179);
}

.delete-history:hover {
  color: #2e2e2e;
}
th {
  width: 4rem;
}
</style>
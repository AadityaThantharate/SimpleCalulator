<template>
  <div class="add-history">
    <h3>Add Calculation History</h3>
    <form @submit.prevent="addHistory">
      <div>
        <label for="expression">Expression:</label>
        <input type="text" id="expression" v-model="expression" required>
      </div>
      <div>
        <label for="result">Result:</label>
        <input type="number" id="result" v-model.number="result" required>
      </div>
      <div>
        <button type="submit">Add History</button>
      </div>
    </form>
  </div>
</template>

<script>
import calculationHistoryService from '@/services/calculationHistoryService';

export default {
  name: 'AddHistory',
  data() {
    return {
      expression: '',
      result: null,
    };
  },
  methods: {
    async addHistory() {
      try {
        await calculationHistoryService.createHistory({
          expression: this.expression,
          result: this.result,
        });

        // Clear the form after successful submission
        this.expression = '';
        this.result = null;
      } catch (error) {
        console.error('Error adding calculation history:', error);
      }
    },
  },
};
</script>

<style scoped>
h3 {
  margin: 40px 0 0;
}

form>div {
  margin: 10px 0;
}

label {
  display: inline-block;
  width: 100px;
}

input {
  padding: 5px;
}

button {
  padding: 5px 10px;
  background-color: #42b983;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}</style>

<template>
  <div id="app">
    <header>
      <h1>Control de Gastos</h1>
    </header>
    <section>
      <div class="info-text">
        <p>Últimos 30 días: {{ last30DaysRange }}</p>
        <p>Saldo Neto: {{ netBalance }}</p>
      </div>
      <form @submit.prevent="addTransaction">
        <input type="text" v-model="description" placeholder="Descripción" required>
        <input type="number" v-model="amount" placeholder="Cantidad" required>
        <select v-model="type">
          <option value="income">Ingreso</option>
          <option value="expense">Gasto</option>
        </select>
        <button type="submit">Agregar</button>
      </form>
      <ul>
        <li v-for="transaction in transactions" :key="transaction.id">
          {{ transaction.description }} - {{ transaction.amount }} - {{ transaction.type === 'income' ? 'Ingreso' : 'Gasto' }}
        </li>
      </ul>
      <div class="chart-container">
        <canvas id="chart"></canvas>
        <div class="balance">Saldo neto: {{ netBalance }}</div>
      </div>
    </section>
  </div>
</template>

<script>
import Chart from 'chart.js/auto';

export default {
  data() {
    return {
      description: '',
      amount: 0,
      type: 'income',
      transactions: [],
      chartInstance: null
    };
  },
  computed: {
    last30DaysRange() {
      const today = new Date();
      const last30Days = new Date(today);
      last30Days.setDate(today.getDate() - 29);
      return `${last30Days.toLocaleDateString()} - ${today.toLocaleDateString()}`;
    },
    netBalance() {
      const incomes = this.transactions.filter(t => t.type === 'income').map(t => t.amount).reduce((a, b) => a + b, 0);
      const expenses = this.transactions.filter(t => t.type === 'expense').map(t => t.amount).reduce((a, b) => a + b, 0);
      return incomes - expenses;
    }
  },
  methods: {
    addTransaction() {
      this.transactions.push({
        id: Date.now(),
        description: this.description,
        amount: parseFloat(this.amount),
        type: this.type
      });
      this.description = '';
      this.amount = 0;
      this.type = 'income';
      this.updateChart();
    },
    updateChart() {
      const incomes = this.transactions.filter(t => t.type === 'income').map(t => t.amount);
      const expenses = this.transactions.filter(t => t.type === 'expense').map(t => t.amount);
      const balance = incomes.reduce((a, b) => a + b, 0) - expenses.reduce((a, b) => a + b, 0);

      const existingChart = this.chartInstance;
      if (existingChart) {
        existingChart.destroy();
      }

      const ctx = document.getElementById('chart').getContext('2d');
      this.chartInstance = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: ['Ingresos', 'Gastos', 'Saldo neto'],
          datasets: [{
            label: 'Control de Gastos',
            data: [incomes.reduce((a, b) => a + b, 0), expenses.reduce((a, b) => a + b, 0), balance],
            backgroundColor: ['#4CAF50', '#F44336', '#2196F3']
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false
        }
      });
    }
  },
  mounted() {
    this.updateChart();
  }
};
</script>

<style src="./assets/app.css"></style>
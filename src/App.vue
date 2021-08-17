<template>
  <main class="current-balance">
    <h1 class="current-balance__main-title">Check your balance!</h1>
    <section class="current-balance__section">
      <h2 class="current-balance__section-title">Current balance</h2>
      <p>Your current balance is {{ currentBalance }}$</p>
      <ul>
        <li v-for="item in formatedExpenses" :key="item.id">
          <p>{{ item.text }}</p>
        </li>
      </ul>
      <template v-if="statistics">
        <h3>Statistics</h3>
        <ul>
          <li v-for="item in statistics" :key="item.category">
            <p>{{ item.category }}: {{ item.value }}</p>
          </li>
        </ul>
      </template>
    </section>
    <section class="current-balance__section">
      <template v-if="currentBalance">
        <h2 class="current-balance__section-title">Spend your money!</h2>
        <form class="current-balance__form" @submit.stop.prevent="handleSubmit">
          <label>
            <p>Total amount:</p>
            <input :value="form.amount" ref="amountInput" type="number" name="totalAmount" autofocus @input="handleAmount" @focus="removeAmountError">
            <p v-if="amountError" class="current-balance__amount-error">{{ amountError }}</p>
          </label>
          <h3>Category</h3>
          <ul class="current-balance__radios-list">
            <li v-for="category in categories" :key="category">
              <label>
                {{ category }}
                <input :value="category" type="radio" name="category" v-model="form.category">
              </label>
            </li>
          </ul>
          <h3>Comment</h3>
          <label>
            <textarea v-model="form.comment"></textarea>
          </label>
          <button>Spend!</button>
        </form>
      </template>
      <template v-else>
        <p>Sorry, you don't have money to spend ='(</p>
        <button @click="clearExpenses">Spend again!</button>
      </template>
    </section>
  </main>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      initialBalance: 1000,
      expenses: [],
      amountError: '',
      initialForm: {
        amount: 0,
        category: 'Housing',
        comment: ''
      },
      form: {
        amount: 0,
        category: 'Housing',
        comment: ''
      },
      categories: ['Housing', 'Food', 'Insurance', 'Transportation', 'Taxes']
    };
  },
  computed: {
    currentBalance() {
      return this.expenses && this.expenses.length ? this.initialBalance - this.expenses.reduce((accumulator, item) => accumulator + +item.amount, 0) : this.initialBalance;
    },
    formatedExpenses() {
      const res = [];
      this.expenses.forEach((i, ind) => {
        const comment = i.comment.trim() ? `(${i.comment.trim()})` : '';
        const template = `You've spent ${+i.amount}$ for ${i.category} ${comment}`.trim();
        res.push({
          text: template,
          id: ind
        });
      });
      return res;
    },
    statistics() {
      const onePercent = this.initialBalance / 100;
      const map = {};
      const res = [];
      this.expenses.forEach(item => {
        if (!map[item.category]) {
          map[item.category] = +item.amount;
        } else {
          map[item.category] += +item.amount
        }
      });
      for (const key of Object.keys(map)) {
        map[key] = `${map[key] / onePercent}%`;
        res.push({
          category: key,
          value: map[key]
        });
      }
      return res.length ? res : null;
    }
  },
  methods: {
    handleAmount(eve) {
      const numbers = /^[0-9]+$/;
      const newVal = eve.target.value;
      const oldVal = this.form.amount;
      if (!newVal.match(numbers) && newVal !== '') {
        setTimeout(() => {
          this.form.amount = oldVal;
        });
      }
      this.form.amount = +newVal;
    },
    handleSubmit() {
      if (!+this.form.amount) return;
      if (+this.form.amount > this.currentBalance) {
        this.amountError = 'You can`t spend more money that you have =('
      } else {
        this.expenses.push(this.form);
        this.form = {...this.initialForm};
        if (this.$refs.amountInput) {
          this.$refs.amountInput.focus();
        }
      }
    },
    removeAmountError() {
      this.amountError = '';
    },
    clearExpenses() {
      this.expenses = [];
    }
  }
}
</script>

<style scoped>
.current-balance {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start;
  width: 100%;
  padding: 30px;
  font-size: 14px;
}
.current-balance__main-title {
  width: 100%;
  font-size: 20px;
}
.current-balance__section {
  width: 50%;
}
.current-balance__section-title {
  font-size: 16px;
}
.current-balance__form {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
.current-balance__radios-list {
  padding: 0;
  list-style-type: none;
}
.current-balance__amount-error {
  padding: 0;
  margin: 0;
  font-size: 10px;
  color: red;
}
</style>
<style>
* {
  box-sizing: border-box;
  color: inherit;
  font-family: inherit;
  line-height: 1.4em;
}
html,
body {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
  font-family: 'Trebuchet MS', sans-serif;
}
</style>

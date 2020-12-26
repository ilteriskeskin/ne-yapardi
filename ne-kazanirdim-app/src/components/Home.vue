<template>
  <div class="container" style="margin-top: 10%">
    <div class="form-group row">
      <label for="example-date-input" class="col-2 col-form-label"
        >Investment Date</label
      >
      <div class="col-4">
        <input
          class="form-control"
          type="date"
          value="2011-08-19"
          id="example-date-input"
          v-model="investment_date"
        />
      </div>
    </div>
    <br />
    <div class="form-group row">
      <label for="example-date-input" class="col-2 col-form-label">Date</label>
      <div class="col-4">
        <input
          class="form-control"
          type="date"
          value="2011-08-19"
          id="example-date-input"
          v-model="date"
        />
      </div>
    </div>
    <br />
    <div class="form-group row">
      <label for="example-date-input" class="col-2 col-form-label">Money</label>
      <div class="col-4">
        <input type="number" placeholder="Number" v-model="money" />
        <select v-model="currency">
          <option v-for="(curr, index) in currencies" :key="index">
            {{ index }}
          </option>
        </select>
      </div>
    </div>
    <br />
    <div class="form-group row">
      <label for="example-date-input" class="col-2 col-form-label"
        >Target Currency</label
      >
      <div class="col-4">
        <select v-model="second_currency">
          <option v-for="(curr, index) in currencies" :key="index">
            {{ index }}
          </option>
        </select>
      </div>
    </div>
    <br />
    <button class="btn btn-primary" @click="calc()">Calc</button>

    <br />
    <br />

    <div v-if="investment_target_currency_value != 0">
      Yatırdığınız Para: <strong>{{ this.money }}</strong> {{ currency }}
      <br />
      Yatırdığınız Paranın {{ investment_date }} Tarihindeki Karşılığı: <strong>
      {{ investment_target_currency_value }} </strong> {{ second_currency }}
      <br />
      Yatırdığınız Paranın {{ date }} Tarihindeki Karşılığı: <strong>
      {{ now_target_currency_value }} </strong> {{ second_currency }}
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "HelloWorld",

  data() {
    return {
      money: 0,
      date: "",
      currency: "",
      second_currency: "",
      rates: {},
      currencies: {},
      investment_date: "",
      investment_result: {},
      now_result: {},
      investment_target_currency_value: 0,
      now_target_currency_value: 0,
    };
  },

  created() {
    var today = new Date();
    var dd = String(today.getDate());
    var mm = String(today.getMonth() + 1);
    var yyyy = today.getFullYear();

    today = yyyy + "-" + mm + "-" + dd;

    let url = "https://api.exchangeratesapi.io/" + today;

    axios.get(url).then((response) => {
      this.currencies = response.data.rates;
    });
  },

  methods: {
    calc() {
      // date format is 2010-01-12
      let investment_url =
        "https://api.exchangeratesapi.io/" +
        this.investment_date +
        "?base=" +
        this.currency;

      if (this.investment_date === "" || this.date === "") {
        alert("Put the dates on !!");
        return 0;
      }

      axios.get(investment_url).then((response) => {
        console.log("Bu yatırım tarihi sorgusu:", response.data);
        this.investment_result = response.data;
        this.calcNow();
      });
    },

    calcNow() {
      let url =
        "https://api.exchangeratesapi.io/" +
        this.date +
        "?base=" +
        this.currency;

      axios.get(url).then((response) => {
        console.log("Bu şimdinin sorgusu:", response.data);
        this.now_result = response.data;
        this.mainCalc();
      });
    },

    mainCalc() {
      this.investment_target_currency_value =
        this.investment_result.rates[this.second_currency] * this.money;
      this.now_target_currency_value =
        this.now_result.rates[this.second_currency] * this.money;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>

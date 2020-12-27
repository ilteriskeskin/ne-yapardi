<template>
  <div class="section">
    <!--<div class="title">
      <h1>Ne Kadar Kazanırım</h1>
    </div>-->

    <div class="main">
      <div class="main-calculate">
        <div class="main-calculate-title">
          <h1>Ne Kadar Kazanırım</h1>
        </div>

        <div class="form-group row m-0">
          <label
            for="example-date-input"
            class="col-12 col-form-label calculate-label"
          >
            Yatırım Tarihi
          </label>
          <div class="col-8">
            <input
              class="form-control calculate-input"
              type="date"
              value="2011-08-19"
              id="example-date-input"
              v-model="investment_date"
            />
          </div>
        </div>

        <br />

        <div class="form-group row m-0">
          <label
            for="example-date-input"
            class="col-12 col-form-label calculate-label"
          >
            Hedef Tarih
          </label>
          <div class="col-8">
            <input
              class="form-control calculate-input"
              type="date"
              value="2011-08-19"
              id="example-date-input"
              v-model="date"
            />
          </div>
        </div>

        <br />

        <div class="form-group row m-0">
          <label
            for="example-date-input"
            class="col-12 col-form-label calculate-label"
          >
            Yatırım Miktarı ve Birimi
          </label>
          <div class="col-8">
            <input
              type="number"
              placeholder="Number"
              v-model="money"
              class="col-10 calculate-input"
            />
            <select v-model="currency" class="calculate-input col-2">
              <option value="default" disabled selected>Birim</option>
              <option v-for="(curr, index) in currencies" :key="index">
                {{ index }}
              </option>
            </select>
          </div>
        </div>

        <br />

        <div class="form-group row m-0">
          <label
            for="example-date-input"
            class="col-12 col-form-label calculate-label"
          >
            Hedef Yatırım Birim
          </label>
          <div class="col-8">
            <select v-model="second_currency" class="col-12 calculate-input">
              <option value="default" disabled selected>Birim</option>
              <option v-for="(curr, index) in currencies" :key="index">
                {{ index }}
              </option>
            </select>
          </div>
        </div>

        <br />

        <button
          class="form-group btn btn-primary calculate-button"
          @click="calc()"
        >
          Hesapla
        </button>

        <br />
        <br />

        <div
          v-if="investment_target_currency_value != 0"
          class="calculate-result"
        >
          <div class="calculate-result-block">
            <span class="calculate-result-block-title"
              >Yatırdığınız Para:
            </span>
            <span class="calculate-result-block-content"
              ><strong>{{ this.money }}</strong> {{ currency }}</span
            >
          </div>

          <div class="calculate-result-block">
            <span class="calculate-result-block-title"
              >Yatırdığınız Paranın {{ investment_date }} Tarihindeki
              Karşılığı:</span
            >
            <span class="calculate-result-block-content"
              >{{ investment_target_currency_value }}
              {{ second_currency }}</span
            >
          </div>

          <div class="calculate-result-block">
            <span class="calculate-result-block-title"
              >Yatırdığınız Paranın {{ date }} Tarihindeki Karşılığı:</span
            >
            <span class="calculate-result-block-content"
              ><strong> {{ now_target_currency_value }} </strong>
              {{ second_currency }}</span
            >
          </div>
        </div>
      </div>
      <div class="main-image">
        <img src="../assets/img/escobar.jpg" />
      </div>
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
      currency: "default",
      second_currency: "default",
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
      var investment_url = "";

      if (
        this.currency === "TRY" &&
        (this.investment_date.substring(0, 4) <= 2005 ||
          this.date.substring(0, 4) <= 2005)
      ) {
        investment_url =
          "https://api.exchangeratesapi.io/" +
          this.investment_date +
          "?base=" +
          "TRL";
      } else {
        investment_url =
          "https://api.exchangeratesapi.io/" +
          this.investment_date +
          "?base=" +
          this.currency;
      }

      // date format is 2010-01-12 // 2005-01-02 -- TRL --> TRY
      if (this.investment_date === "" || this.date === "") {
        alert("Put the inputs on !!");
        return 0;
      }

      axios.get(investment_url).then((response) => {
        this.investment_result = response.data;
        this.calcNow();
      });
    },

    calcNow() {
      var url = "";

      if (
        this.currency === "TRY" &&
        (this.investment_date.substring(0, 4) <= 2005 ||
          this.date.substring(0, 4) <= 2005)
      ) {
        url = "https://api.exchangeratesapi.io/" + this.date + "?base=" + "TRL";
      } else {
        url =
          "https://api.exchangeratesapi.io/" +
          this.date +
          "?base=" +
          this.currency;
      }

      axios.get(url).then((response) => {
        this.now_result = response.data;
        this.mainCalc();
      });
    },

    mainCalc() {
      if (isNaN(this.investment_result.rates[this.second_currency])) {
        this.investment_target_currency_value =
          this.investment_result.rates["TRL"] * this.money;

        if (isNaN(this.now_result.rates[this.second_currency])) {
          this.now_target_currency_value =
            this.now_result.rates["TRL"] * this.money;
        } else if (this.now_result.rates[this.second_currency]) {
          this.now_target_currency_value =
            this.now_result.rates[this.second_currency] * this.money;
        }
      } else {
        this.investment_target_currency_value =
          this.investment_result.rates[this.second_currency] * this.money;

        this.now_target_currency_value =
          this.now_result.rates[this.second_currency] * this.money;
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
/* 
darkColor          : #1A1F22;
lightBackground    : #F4F7FA;
fixShadow          : 0px 0px 1px 1px rgba(0,0,0,0.10);
*/

* {
  font-family: "Oxygen", sans-serif;
}

.section {
  padding-left: 100px;
}

.title {
  height: 15vh;
  width: 100%;
  display: flex;
  align-items: flex-end;
}

.main {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-wrap: nowrap;
  justify-content: center;
  align-items: center;
}

.main-calculate {
  width: 50%;
  height: auto;
}

.main-calculate-title {
  padding-left: 12px;
  height: 10vh;
}

.main-calculate-title h1 {
  font-weight: 700;
  color: #333;
}

.calculate-label {
  font-size: 16px;
  font-weight: 600;
  color: gray;
}

.main-image {
  width: 50%;
  height: 100vh;
  object-fit: cover;
}

.main-image img {
  height: 100vh;
  width: 100%;
  object-fit: cover;
  -webkit-box-shadow: 0px 0px 15px 5px rgba(0, 0, 0, 0.4);
  box-shadow: 0px 0px 15px 5px rgba(82, 64, 64, 0.4);
}

.calculate-input {
  appearance: none;
  padding: 15px 20px;
  box-shadow: 0px 0px 1px 1px rgba(0, 0, 0, 0.1);
  border: none;
  background-color: #f4f7fa;
  border-radius: 4px;
  color: #999;
  outline: none;
}

.calculate-button {
  background-color: #38c172;
  margin-left: 12px;
  border: none;
  padding: 15px 40px;
  font-weight: 500;
  outline: none;
}

.calculate-result-block {
  width: 65%;
  height: auto;
  display: flex;
  flex-wrap: nowrap;
  justify-content: space-between;
  align-items: center;
  padding-block: 20px;
  box-shadow: 0px 0px 1px 1px rgba(0, 0, 0, 0.05);
  margin-left: 12px;
  padding-left: 10px;
  margin-bottom: 15px;
  border-radius: 4px;
  background-color: #38c17220;
}

.calculate-result-block-title {
  width: 50%;
  height: auto;
}

.calculate-result-block-content {
  width: 50%;
  height: auto;
}

@media only screen and (max-width: 1150px) {
  .main-image {
    display: none;
  }
  .main-calculate {
    width: 100%;
    height: 100%;
  }
  .calculate-result {
    width: 100%;
  }
}
</style>

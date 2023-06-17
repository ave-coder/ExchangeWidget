<template>
  <div class="container max-w-screen-xl mx-auto flex flex-col text-left">
    <div class="max-w-2xl mx-auto">
      <h2 class="text-2xl sm:text-5xl font-bold mb-4 text-center sm:text-left"> Crypto Exchange </h2>
      <p class='text-base sm:text-xl mb-6 text-center sm:text-left'>
        <span>Exchange fast and easy</span>
      </p>
      <div class="exchange-widget__pair-selector flex flex-col sm:flex-row items-center justify-center sm:flex-nowrap">
        <div class="flex items-stretch w-full mb-4 sm:mb-0 sm:w-auto sm:mr-4">
          <input class='border border-solid border-gray-200 bg-gray-100 w-full sm:w-auto mr-0 px-4 py-2' type="number"
            v-model="userValue" @change="setEstimatedAmount" />
          <SelectForm :value="fromCurrency" :list="currencies" @change-option="setFromCurrencies"
            class="w-full sm:w-auto mt-2 sm:mt-0">
          </SelectForm>
        </div>
        <div class="flex flex-col justify-center cursor-pointer mt-3 sm:mt-0 sm:ml-3">
          <img src="../img/Vector.png" class="mx-8" alt="" @click="switchValue" />
          <img src="../img/Vector.png" class="mx-8 transform rotate-180" alt="" @click="switchValue" />
        </div>
        <div class="flex items-stretch w-full sm:w-auto flex-grow-0 sm:justify-center">
          <input class='border border-solid border-gray-200 bg-gray-100 py-2 w-full sm:w-auto ml-0 sm:ml-4 px-4'
            type="text" :value="estimatedValue" readonly />
          <SelectForm :value="toCurrency" :list="currencies" @change-option="setToCurrencies"
            class="w-full sm:w-auto mt-2 sm:mt-0 mr-4">
          </SelectForm>
        </div>
      </div>
      <div class="mt-8">
        <p class='mb-4 text-base text-center sm:text-left'>Your Ethereum address</p>
        <div class="flex flex-col items-center sm:flex-row">
          <input
            class="border border-solid bg-gray-100 h-10 rounded-md py-2 px-4 w-full mb-4 sm:mb-0 sm:mr-4 sm:w-auto sm:w-1/2 lg:w-2/3 xl:w-3/4 2xl:w-4/5"
            type="text" />
          <button
            class="text-base sm:text-lg uppercase bg-sky-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded-md w-full sm:w-1/4 lg:w-1/3 xl:w-1/4 2xl:w-1/5 mt-2 sm:mt-0 flex justify-center items-center">
            Exchange
          </button>
        </div>
        <p v-if="minAmount === null || estimatedValueRaw === null || estimatedValue === '-'"
          class="text-red-600 mt-4 text-center"> This pair is disabled now </p>
      </div>
      <div class="mt-5">
        <input v-model="searchText" @input="searchCurrencies" type="text" placeholder="Search"
          class="w-full border-2 border-gray-300 bg-white h-10 px-5 pr-10 rounded-lg text-sm focus:outline-none focus:border-sky-500"
          @focus="showList = true" />
        <div class="absolute right-0 top-0 mt-3 mr-4">
          <button @click="toggleList" type="button"
            class="text-gray-100 text-gray-300 focus:outline-none focus:text-gray-400">
            <svg class="h-4 w-4 fill-current" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
              <path :class="{'hidden': showList}" fill-rule="evenodd" clip-rule="evenodd"
                d="M14.325 16L12 13.675 9.675 16 8 14.325 10.325 12 8 9.675 9.675 8 12 10.325 14.325 8 16 9.675 13.675 12 16 14.325 14.325z">
              </path>
              <path :class="{'hidden': !showList}" fill-rule="evenodd" clip-rule="evenodd"
                d="M9.675 8L12 10.325 14.325 8 16 9.675 13.675 12 16 14.325 14.325 16 12 13.675 9.675 16 8 14.325 10.325 12 8 9.675z">
              </path>
            </svg>
          </button>
        </div>
        <div v-if="showList" class="mt-1 bg-white rounded-lg shadow-lg z-10">
          <div v-for="(currency, index) in filteredCurrencies" :key="index" @click="selectCurrency(currency)"
            class="px-4 py-2 hover:bg-sky-300 hover:text-white cursor-pointer"> {{ currency.name }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

  import SelectForm from './SelectForm.vue';

  export default {
    data() {
      return {
        currencies: [], // массив валют, получаемый из API
        fromCurrency: '', // выбранная отправная валюта
        toCurrency: '', // выбранная конечная валюта
        minAmount: undefined, // минимальная сумма обмена, устанавливаемая из API
        userValue: null, // значение, введенное пользователем для обмена
        estimatedValueRaw: undefined, // предварительно рассчитанная сумма получаемой валюты для обмена
        searchText: '', // строка поиска на странице выбора валют
        showList: false, // флаг, указывающий, отображать ли выпадающий список валют
        filteredCurrencies: [] // массив отфильтрованных валют в зависимости от строки поиска
      }
    },
    components: {
      SelectForm // импорт компонента формы выбора валют
    },
    name: 'ExchangeWidget',
    methods: {

      async getCurrencies() { // метод для получения доступных валют из API
        const response = await fetch('https://api.changenow.io/v1/currencies?active=true'); // запрос к API
        const data = await response.json(); // обработка ответа в формат JSON
        this.currencies = data.map(currency => { // маппинг массива данных, полученных из API, на массив объектов с выбранными свойствами
          return {
            name: currency.name, // название валюты
            ticker: currency.ticker, // тикер валюты
            image: currency.image, // изображение валюты
          };
        });
      },

      async setMinAmount() { // установить минимальную сумму обмена между выбранными валютами из API
        const API_KEY = 'c9155859d90d239f909d2906233816b26cd8cf5ede44702d422667672b58b0cd'; // ключ API
        let apiUrl = `https://api.changenow.io/v1/min-amount/${this.fromCurrency.ticker}_${this.toCurrency.ticker}?api_key=${API_KEY}`; // адрес API запроса с выбранными валютами

        fetch(apiUrl) // GET-запрос к API
          .then(response => response.json()) //обработка ответа в формат JSON
          .then(data => {
            this.minAmount = data.minAmount; // установка минимальной суммы обмена из ответа API
            this.userValue = data.minAmount; // установка введенного пользователем значения, равного минимальной сумме обмена
          })
          .catch(error => console.log(error)) // вывод ошибки в консоль
          .finally(this.setEstimatedAmount) // вызов функции для установки предварительно рассчитанной суммы получаемой валюты.
      },

      async setEstimatedAmount() { // функция для рассчета предварительной суммы получаемой валюты для обмена
        const API_KEY = 'c9155859d90d239f909d2906233816b26cd8cf5ede44702d422667672b58b0cd'; // ключ API
        let apiUrl = `https://api.changenow.io/v1/exchange-amount/${this.userValue}/${this.fromCurrency.ticker}_${this.toCurrency.ticker}?api_key=${API_KEY}`; // запрос к API с выбранными валютами и пользовательским значением

        fetch(apiUrl) // GET-запрос к API
          .then(response => response.json()) //обработка ответа в формат JSON
          .then(data => {
            this.estimatedValueRaw = data.estimatedAmount; // установка предварительно рассчитанной суммы получаемой валюты
          })
          .catch(error => console.log(error)); // вывод ошибки в консоль
      },

      async setFromCurrencies(currency) { // функция, которая устанавливает отправную валюту при выборе пользователем валюты из выпадающего списка
        this.fromCurrency = currency; // установка выбранной отправной валюты
        if (this.fromCurrency && this.toCurrency) await this.setMinAmount(); // вызов функции установки минимальной суммы обмена, если обе валюты были выбраны
      },

      async setToCurrencies(currency) { // функция, которая устанавливает конечную валюту при выборе пользователем валюты из выпадающего списка
        this.toCurrency = currency; // установка выбранной конечной валюты
        if (this.fromCurrency && this.toCurrency) await this.setMinAmount(); // вызов функции установки минимальной суммы обмена, если обе валюты были выбраны
      },

      async switchValue() {
        // Сохраняем значение левого инпут в переменную
        const leftValue = this.userValue
        // Заменяем значение левого инпут на значение правого инпута
        this.userValue = this.estimatedValueRaw
        // Заменяем значение правого инпута на значение левого инпута
        this.estimatedValueRaw = leftValue
        // Обновляем оценочное значение
        this.setEstimatedAmount()
      },

      async toggleList() {
        this.showList = !this.showList // переключатель флага showList
      },

      async searchCurrencies() {
        this.filteredCurrencies = this.currencies.filter(currency =>
          currency.name.toLowerCase().includes(this.searchText.toLowerCase())
        ) // фильтрует валюты на основе введенного пользователем текста в строку поиска и сохраняет результат в массив filteredCurrencies
      },

      async selectCurrency(currency) {
        this.searchText = currency.name // устанавливает текст строки поиска равным названию выбранной валюты
        this.showList = false // скрывает выпадающий список валют
      },
    },

    computed: {
      estimatedValue: {
        get() {
          return this.userValue < this.minAmount ? '-' : this.estimatedValueRaw; // предварительно рассчитанная сумма получаемой валюты, если пользовательский ввод достаточен, иначе "-"
        }
      },
    },
    mounted() {
      this.getCurrencies(); // вызов функции получения доступных валют на момент запуска компонента
    }
  }
</script>

<style scoped>
  * {
    font-family: 'Vollkorn', serif;
  }

  .text-2xl {
    font-size: 40px;
  }

  .text-base {
    font-size: 20px;
  }

  .rotate-90 {
    transform: rotate(90deg);
  }

  @media(max-width: 640px) {
    .exchange-widget__pair-selector {
      flex-direction: column;
      justify-content: center;
      items-center: center;
    }

    .exchange-widget__pair-selector>div {
      margin: 0 auto;
      max-width: 4xl;
    }
  }
</style>
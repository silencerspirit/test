<template>
  <v-app>
    <div class="container">
      <h1 class="app-title">Проверка выражения на конъюнктивность/дизъюнктивность</h1>
      <v-divider class="my-4" />
      <v-form class="form" v-model="isValidForm" @submit.prevent="resolveExpression">
        <v-text-field
          class="form__input"
          v-model="expression"
          placeholder="Ввидите число от 0 до 1"
          :rules="rules"
          :error-messages="incorrectExpression ? 'Выражение некорректно' : ''"
          @input="clear"
        ></v-text-field>
        <div class="form__operators">
          <v-btn
            class="form__btn"
            small
            fab
            color="primary"
            @click="expression += '0'"
          >0</v-btn>
          <v-btn
            class="form__btn"
            small
            fab
            color="primary"
            @click="expression += '1'"
          >1</v-btn>
          <v-tooltip top>
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                class="form__btn"
                small
                fab
                color="warning"
                v-bind="attrs"
                v-on="on"
                @click="expression += '&'"
              >&</v-btn>
            </template>
            <span>Логическое И</span>
          </v-tooltip>
          <v-tooltip top>
            <template v-slot:activator="{ on, attrs }">
              <v-btn
                class="form__btn"
                small
                fab
                color="warning"
                v-bind="attrs"
                v-on="on"
                @click="expression += 'v'"
              >v</v-btn>
            </template>
            <span>Логическое ИЛИ</span>
          </v-tooltip>
          <v-btn
            :disabled="!isValidForm"
            type="submit"
            class="form__submit"
            rounded
            color="primary"
            >Решить</v-btn>
        </div>
        <div class="form__result">
          <h3>
            Конъюнктивность:
            <v-icon :color="isConjunction ? 'green' : 'dark'">mdi-check</v-icon>
          </h3>
          <h3>
            Дизъюнктивность:
            <v-icon :color="isDisjunction ? 'green' : 'dark'">mdi-check</v-icon>
          </h3>
          <h3>Результат: {{result}}</h3>
        </div>
      </v-form>
    </div>
  </v-app>
</template>

<script>
export default {
  name: 'App',

  data: () => ({
    expression: '',
    isValidForm: false,
    isConjunction: false,
    isDisjunction: false,
    result: '-',
    incorrectExpression: false,
  }),

  computed: {
    isLastChar() {
      const lastValue = this.expression[this.expression.length - 1];
      return this.validCharacters.includes(lastValue) || !this.expression;
    },
    incorrectOrder() {
      let incorrectOrder = false;
      this.expression.split('').forEach((item, i) => {
        const condition = this.validValues.includes(this.expression[i])
        && this.validValues.includes(this.expression[i + 1]);
        if (condition) incorrectOrder = true;
      });
      return incorrectOrder;
    },
  },

  created() {
    this.validValues = ['0', '1'];
    this.validCharacters = ['&', 'v'];
    this.rules = [
      (v) => !!v || 'Введите выражение',
      (v) => v.length > 2 || 'Ввидите как минимум два числа и добавьте оператор',
      (v) => v.split('').every((_) => [...this.validValues, ...this.validCharacters].includes(_)) || `Введите допустымые значения: ${[...this.validValues, ...this.validCharacters].join(', ')}`,
    ];
  },

  methods: {
    resolveExpression() {
      if (this.isLastChar || this.incorrectOrder) {
        this.incorrectExpression = true;
        return;
      }
      const mathResult = this.expression.split('').map((item) => {
        if (item === '&') return '*';
        if (item === 'v') return '+';
        return item;
      }).join('');
      this.isConjunction = this.expression.split('').every((item) => ['1', '&'].includes(item));
      try {
        this.isDisjunction = !!eval(mathResult);
        this.result = eval(mathResult);
      } catch (error) {
        this.incorrectExpression = true;
      }
    },
    clear() {
      this.incorrectExpression = false;
      this.isDisjunction = false;
      this.isConjunction = false;
      this.result = '-';
    },
  },
};
</script>

<style lang="scss">
body {
  padding: 24px 0;
  text-align: center;
}
.container {
  min-width: 320px;
  padding: 0 16px;
  max-width: 800px;
}
.form {
  display: flex;
  flex-wrap: wrap;
  text-align: left;
  &__operators {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    width: 100%;
    max-width: 260px;
    margin: 8px 0 0 0
  }
  &__btn {
    margin:0 8px 0 0;
  }
  &__submit {
    text-transform: unset !important;
    width: 100%;
    margin: 12px 0 0 0;
  }
  &__input {
    width: 100%;
  }
  &__result {
    display: flex;
    flex-direction: column;
    margin-left: 24px;
    h3 {
      margin-bottom: 8px;
    }
  }
}
  @media (max-width: 767px) {
    .app-title {
      font-size: 24px;
    }
  }

  @media (max-width: 520px) {
    .form {
      flex-direction: column;
      &__operators {
        margin: 0 auto;
      }
      &__result {
        margin: 16px 0;
      }
      &__submit {
        margin-top: 24px;
      }
    }
  }
</style>

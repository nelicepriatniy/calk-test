<template>
  <form @submit.prevent="handleSubmit">
    <div class="form-wrapper" v-for="(inp, index) in serverData" :key="index">
      <label class="num-wrapper" v-if="inp.type === 'number'">
        <p>{{ inp.name }} <span v-if="inp.isReqire">*</span></p>
        <input
          type="number"
          :class="{ red: inp.isRed }"
          :name="inp.name"
          :min="inp.min"
          :max="inp.max"
          v-model="inp.value"
        />
      </label>

      <label class="select-wrapper" v-else-if="inp.type === 'select'">
        <p>{{ inp.name }} <span v-if="inp.isReqire">*</span></p>
        <select :name="inp.name" v-model="inp.value" :class="{ red: inp.isRed }">
          <option v-for="(opt, index) in inp.values" :key="index" :value="opt">
            {{ opt }}
          </option>
        </select>
      </label>
    </div>

    <button type="submit">Submit</button>

    <div class="mess" v-if="formData.state">
      <div v-if="formData.sucses.state">{{ formData.sucses.message }} {{ formData.result }}</div>
      <div v-if="formData.error.state">{{ formData.error.message }}</div>
    </div>
  </form>
</template>

<script>
export default {
  data() {
    return {
      formData: {
        result: '',
        state: false,
        sucses: {
          message: 'форма отправлена успешно (data вывел в консоль)',
          state: false,
        },
        error: {
          message: 'форма не отправлена',
          state: false,
        },
      },
      serverData: {
        0: {
          name: 'num1',
          type: 'number',
          min: 1,
          max: 19,
          startValue: 1,
          isReqire: false,
          value: '',
          isRed: false,
        },
        1: {
          name: 'sel1',
          type: 'select',
          isReqire: false,
          value: '',
          isRed: false,
          startValue: 0,
          isSelected: true,
          values: {
            0: '+',
            1: '-',
            2: '*',
          },
        },
        3: {
          name: 'num2',
          type: 'number',
          min: 0,
          max: 19,
          value: '',
          isReqire: true,
          isRed: false,
          startValue: 5,
        },
        4: {
          name: 'sel2',
          type: 'select',
          isReqire: true,
          value: '',
          startValue: 1,
          isRed: false,
          values: {
            0: '+',
            1: '-',
            2: '*',
          },
        },
        5: {
          name: 'num3',
          type: 'number',
          min: 0,
          max: 19,
          value: '',
          isReqire: true,
          isRed: false,
          startValue: 8,
        },
      },
    }
  },
  mounted() {
    for (const inp of Object.values(this.serverData)) {
      if (inp.type === 'number' && inp.value === '') {
        inp.value = inp.startValue !== undefined ? inp.startValue : ''
      } else if (inp.type === 'select' && inp.value === '') {
        const firstKey = Object.keys(inp.values)[inp.startValue]
        inp.value = inp.values[firstKey] || ''
      }
    }
  },

  methods: {
    handleSubmit() {
      const result = {}
      const expressionTokens = []

      for (const inp of Object.values(this.serverData)) {
        const val = inp.value

        if (
          (inp.isReqire && val === '') ||
          (inp.type === 'number' && (val > inp.max || val < inp.min))
        ) {
          inp.isRed = true
          this.formData.state = true
          this.formData.error.state = true
          this.formData.sucses.state = false
          return
        } else {
          inp.isRed = false
          result[inp.name] = val
          expressionTokens.push(val)
        }
      }

      const tokens = expressionTokens.map((token) => {
        if (!isNaN(token) && token !== '') return Number(token)
        return token
      })

      let calcResult
      try {
        calcResult = this.calculateExpression(tokens)
      } catch (e) {
        this.formData.state = true
        this.formData.error.state = true
        this.formData.sucses.state = false
        this.formData.error.message = e.message
        return
      }

      this.formData.state = true
      this.formData.error.state = false
      this.formData.sucses.state = true

      console.log('data:', result)
      console.log('=:', calcResult)
      this.formData.result = calcResult
    },
    calculateExpression(tokens) {
      let stack = []
      let i = 0

      while (i < tokens.length) {
        if (tokens[i] === '*') {
          const prev = stack.pop()
          const next = Number(tokens[i + 1])
          let res = 0
          res = prev * next
          stack.push(res)
          i += 2
        } else {
          stack.push(tokens[i])
          i++
        }
      }

      // Теперь складываем и вычитаем
      let result = Number(stack[0])
      i = 1
      while (i < stack.length) {
        const num = Number(stack[i + 1])
        if (stack[i] === '+') result += num
        else if (stack[i] === '-') result -= num
        i += 2
      }

      return result
    },
  },
}
</script>

<style scoped>
form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}
input,
select {
  width: 500px;
  background: none;
  border: none;
  border-bottom: 1px solid white;
  color: white;
  padding: 10px;
}

input.red,
select.red {
  border: 1px solid red;
}
</style>

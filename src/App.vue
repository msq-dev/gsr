<template>
  <div id="app">
    <div class="calculator">

      <InputComponent v-for="input in inputs"
        :key="input.name"
        :ref="input.name"
        :hasInfo="input.hasInfo"
        :info="input.name"
        :label="input.label.replace('Freibetrag', 'Rechtsform')"
        :unit="input.unit"
        :selectableInput="input.selectableInput"
        :selectOptions="input.selectOptions"
        v-model="$data[input.name]"
        @input="handleNewInput"
        @validate="validate"
      />

      <transition name="slide">
        <div v-if="profitMissing" class="warning">Bitte geben Sie Ihren Gewinn an</div>
        <div v-if="invalidInput" class="warning">Bitte überprüfen Sie Ihre Eingabe</div>
      </transition>

      <button class="btn calc-btn" @click="validate">Berechnen</button>

      <transition name="slide">
        <div v-if="calculated && !invalidInput && !profitMissing" class="output-block">

          <OutputComponent v-for="output in outputs"
            :key="output.label"
            :value="$data[output.name]"
            :label="output.label"
            :unit="output.unit"
            :operator="output.operator"
            :labelClass="output.labelClass"
          />

        </div>
      </transition>

    </div>
  </div>

</template>

<script>
import InputComponent from './components/InputComponent.vue'
import OutputComponent from './components/OutputComponent.vue'

function cleanFloat(input) {
  input = input !== "" ? input : "0"
  input = input.replaceAll(",", ".")
  return parseFloat(input)
}

function removeWhitespace(s) {
  return s.replace(/\s+/g, "")
}

export default {
  name: 'App',
  components: {
    InputComponent,
    OutputComponent
  },
  data() {
    return {
      inputs: [
        {
          name: "gewinn",
          value: this.gewinn,
          unit: "€",
          label: "Gewinn aus Gewerbebetrieb",
        },
        {
          name: "hinzurechnung",
          value: this.hinzurechnung,
          unit: "€",
          hasInfo: true,
          label: "Hinzurech&shy;nungsbetrag",
        },
        {
          name: "kuerzung",
          value: this.kuerzung,
          unit: "€",
          hasInfo: true,
          label: "Kürzungs&shy;betrag",
        },
        {
          name: "hebesatz",
          value: this.hebesatz,
          unit: "%",
          hasInfo: true,
          label: "Hebesatz",
        },
        {
          name: "freibetrag",
          value: this.freibetrag,
          hasInfo: true,
          label: "Freibetrag",
          selectableInput: true,
          selectOptions: [
            {
              rechtsform: "Einzelunternehmen",
              freibetrag: 24500
            },
            {
              rechtsform: "Personengesellschaft",
              freibetrag: 24500
            },
            {
              rechtsform: "Kapitalgesellschaft",
              freibetrag: 0
            },
            {
              rechtsform: "rechtsfähiger Verein",
              freibetrag: 5000
            },
          ]
        }
      ],
      outputs: [
        {
          name: "gewinn",
          label: "Gewinn aus Gewerbe",
          unit: "€",
        },
        {
          name: "hinzurechnung",
          label: "Hinzurech&shy;nungs&shy;betrag",
          unit: "€",
          operator: "&plus;",
        },
        {
          name: "kuerzung",
          label: "Kürzungsbetrag",
          unit: "€",
          operator: "&minus;",
        },
        {
          name: "freibetrag",
          label: "Freibetrag",
          unit: "€",
          operator: "&minus;",
          labelClass: "last-calc",
        },
        {
          name: "gewerbeErtrag",
          label: "Maßgebender Gewerbeertrag",
          unit: "€",
        },
        {
          name: "steuerMessZahl",
          label: "Gewerbest.-Messzahl",
          unit: "%",
          operator: "&times;",
          labelClass: "last-calc"
        },
        {
          name: "steuerMessBetrag",
          label: "Steuer&shy;mess&shy;betrag",
          unit: "€",
          operator: "&equals;"
        },
        {
          name: "hebesatz",
          label: "Hebesatz",
          unit: "%",
          operator: "&times;",
          labelClass: "last-calc",
        },
        {
          name: "gewerbeSteuer",
          label: "Gewerbe&shy;steuer",
          unit: "€",
          labelClass: "final-result",
        },
      ],

      gewinn: "",
      hinzurechnung: "",
      kuerzung: "",
      hebesatz: "200",
      freibetrag: "",

      profitMissing: false,
      invalidInput: false,
      calculated: false,

      gewerbeErtrag: 0,
      steuerMessBetrag: 0,
      steuerMessZahl: 3.5,
      gewerbeSteuer: 0
    }
  },

  methods: {
    handleNewInput() {
      if (this.calculated) {
        this.calculated = false
      }
    },

    validate() {
      this.profitMissing = false
      this.invalidInput = false

      const inputComponents = Object.entries(this.$refs)
      const re = /^\d*[,.]?\d*$/

      inputComponents.forEach(component => {
        const inputField = component[1][0].$refs.input || null
        inputField.classList.remove("invalid")

        if (!re.test(removeWhitespace(this.$data[component[0]]))) {
          inputField.focus()
          inputField.classList.add("invalid")

          this.invalidInput = true
        }
      })

      if (!this.gewinn) {
        this.$refs["gewinn"][0].$refs.input.focus()
        this.$refs["gewinn"][0].$refs.input.classList.add("invalid")

        this.profitMissing = true
      }

      if (!this.invalidInput && !this.profitMissing) {
        this.calculate()
      }
    },

    calculate() {
      this.gewinn = removeWhitespace(this.gewinn)
      this.hinzurechnung = removeWhitespace(this.hinzurechnung)
      this.kuerzung = removeWhitespace(this.kuerzung)
      this.hebesatz = this.hebesatz ? removeWhitespace(this.hebesatz) : "200"

      this.gewerbeErtrag =
          cleanFloat(this.gewinn)
        + cleanFloat(this.hinzurechnung)
        - cleanFloat(this.kuerzung)
        - cleanFloat(this.freibetrag)

      this.steuerMessBetrag = Math.floor(this.gewerbeErtrag * 0.035)
      this.gewerbeSteuer = this.steuerMessBetrag * (cleanFloat(this.hebesatz) / 100)

      this.gewerbeErtrag = this.gewerbeErtrag > 0 ? this.gewerbeErtrag : 0
      this.steuerMessBetrag = this.steuerMessBetrag > 0 ? this.steuerMessBetrag : 0
      this.gewerbeSteuer = this.gewerbeSteuer > 0 ? this.gewerbeSteuer : 0

      this.calculated = true
    },
  }
}
</script>

<style>
/* @import url('https://fonts.googleapis.com/css2?family=Azeret+Mono:wght@300&family=Encode+Sans:wght@300&family=Open+Sans:wght@300&display=swap'); */

*,
*::before,
*::after {
  box-sizing: border-box;
}

#app {
  --clr-text: rgba(74, 74, 74);
  --clr-accent: rgb(20, 147, 190);
  --clr-warning: tomato;

  display: grid;
  place-content: center;
  font-family: "Open Sans", sans-serif;
  color: var(--clr-text);
  height: fit-content;
  margin-top: 1rem;
}

.calculator {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.warning {
  color: var(--clr-warning);
  font-weight: bold;
  margin-bottom: 1rem;
}

.btn {
  padding: 1rem 2rem;
  font-family: inherit;
  font-size: inherit;
  color: #fff;
  background: var(--clr-accent);
  border: none;
  border-radius: 9999px;
  cursor: pointer;
}

.calc-btn {
  margin-top: 1rem;
}

.output-block {
  margin-top: 3rem;
}

.slide-enter-active {
  transition: opacity 0.3s, max-height 0.5s;
}

.slide-leave-active {
  transition: opacity 0.05s, max-height 0.3s;  
}

.slide-enter, .slide-leave-to {
  max-height: 0;
  opacity: 0;
}

</style>

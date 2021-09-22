<template>
  <div id="app">
    <div class="calculator">

      <InputComponent
        ref="gewinn"
        label="Gewinn aus Gewerbebetrieb"
        unit="€"
        v-model="gewinn"
        @validate="validate()"
      />
      <InputComponent
        ref="hinzurechnung"
        label="Hinzurech&shy;nungs&shy;betrag"
        unit="€"
        v-model="hinzurechnung"
        @validate="validate()"
      />
      <InputComponent
        ref="kuerzung"
        label="Kürzungsbetrag"
        unit="€"
        v-model="kuerzung"
        @validate="validate()"
      />
      <InputComponent
        ref="hebesatz"
        label="Hebesatz"
        unit="%"
        v-model="hebesatz"
        @validate="validate()"
      />

      <div class="legal-input">
        <label for="rechtsform">Rechtsform</label>
        <select id="rechtsform" v-model="rechtsform">
          <option disabled value>Bitte auswählen</option>
          <option>Einzelunternehmen</option>
          <option>Personengesellschaft</option>
          <option>Kapitalgesellschaft</option>
          <option>rechtsfähiger Verein</option>
        </select>
      </div>

      <div v-if="profitMissing" class="warning">Bitte geben Sie Ihren Gewinn an</div>
      <div v-if="invalidInput" class="warning">Bitte überprüfen Sie Ihre Eingabe</div>

      <button class="calc-btn" @click="validate()">Berechnen</button>
      
      <div v-if="calculated && !invalidInput && !profitMissing" class="output-block">

        <OutputComponent
          :value="gewinn"
          label="Gewinn aus Gewerbe"
          unit="€"
          :clean="false"
        />
        <OutputComponent
          :value="hinzurechnung"
          v-model="hinzurechnung"
          label="Hinzurech&shy;nungs&shy;betrag"
          unit="€"
          operator="&plus;"
          :clean="false"
        />
        <OutputComponent
          :value="kuerzung"
          label="Kürzungsbetrag"
          unit="€"
          operator="&minus;"
          :clean="false"
        />
        <OutputComponent
          :value="String(freibetrag)"
          label="Freibetrag"
          unit="€"
          operator="&minus;"
          :clean="true"
          labelClass="last-calc"
        />
        <OutputComponent
          :value="String(gewerbeErtrag)"
          label="Maßgebender Gewerbeertrag"
          unit="€"
          :clean="true"
        />
        <OutputComponent
          value="3,50"
          label="Gewerbest.-Messzahl"
          unit="%"
          operator="&times;"
          labelClass="last-calc"
        />
        <OutputComponent
          :value="String(steuerMessBetrag)"
          label="Steuer&shy;mess&shy;betrag"
          unit="€"
          :clean="true"
        />
        <OutputComponent
          :value="hebesatz"
          label="Hebesatz"
          unit="%"
          operator="&times;"
          :clean="false"
          labelClass="last-calc"
        />
        <OutputComponent
          :value="String(gewerbeSteuer)"
          label="Gewerbe&shy;steuer"
          unit="€"
          :clean="true"
          labelClass="final-result"
        />
      </div>
    </div>
  </div>

</template>

<script>
import InputComponent from './components/InputComponent.vue'
import OutputComponent from './components/OutputComponent.vue'

const freiBetraege = {
  "Einzelunternehmen": 24500,
  "Personengesellschaft": 24500,
  "Kapitalgesellschaft": 0,
  "rechtsfähiger Verein": 5000
}

function cleanNumber(input) {
  if (input === "") { input = "0" }
  input = input.replaceAll(" ", "")
  input = input.replaceAll(",", ".")
  return parseFloat(input)
}

export default {
  name: 'App',
  components: {
    InputComponent,
    OutputComponent
  },
  data() {
    return {
      gewinn: "",
      hinzurechnung: "",
      kuerzung: "",
      hebesatz: "200",
      rechtsform: "",
      profitMissing: false,
      invalidInput: false,
      calculated: false,
      freibetrag: 0,
      gewerbeErtrag: 0,
      steuerMessBetrag: 0,
      gewerbeSteuer: 0,
    }
  },
  methods: {
    validate() {
      this.profitMissing = false
      this.invalidInput = false

      const fieldsToValidate= [
        {
          value: this.gewinn,
          inputField: this.$refs.gewinn.$refs.input
        },
        {
          value: this.hinzurechnung,
          inputField: this.$refs.hinzurechnung.$refs.input
        },
        {
          value: this.kuerzung,
          inputField: this.$refs.kuerzung.$refs.input
        },
        {
          value: this.hebesatz,
          inputField: this.$refs.hebesatz.$refs.input
        }
      ]

      const re = /^\d*[,.]?\d*$/

      fieldsToValidate.forEach(field => {
        field.inputField.classList.remove("invalid")
        if (!re.test(field.value.replaceAll(" ", ""))) {
          field.inputField.classList.add("invalid")
          this.invalidInput = true
        }
      })

      if (!this.gewinn) {
        this.$refs.gewinn.$refs.input.classList.add("invalid")
        this.profitMissing = true
      }

      if (!this.invalidInput && !this.profitMissing) {
        this.gewinn = this.gewinn.replaceAll(" ", "")
        this.hinzurechnung = this.hinzurechnung.replaceAll(" ", "")
        this.kuerzung = this.kuerzung.replaceAll(" ", "")
        this.hebesatz = this.hebesatz.replaceAll(" ", "")
        this.calculate()
      }

    },
    calculate() {
      this.freibetrag = freiBetraege[this.rechtsform] ? freiBetraege[this.rechtsform] : 0
      this.hebesatz = this.hebesatz ? this.hebesatz : "200"

      this.gewerbeErtrag =
          cleanNumber(this.gewinn)
        + cleanNumber(this.hinzurechnung)
        - cleanNumber(this.kuerzung)
        - this.freibetrag

      this.steuerMessBetrag = Math.floor(this.gewerbeErtrag * 0.035)
      this.gewerbeSteuer = this.steuerMessBetrag * (cleanNumber(this.hebesatz) / 100)

      this.calculated = true
    },
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Azeret+Mono:wght@300&family=Encode+Sans:wght@300&family=Open+Sans:wght@300&display=swap');

*,
*::before,
*::after {
  box-sizing: border-box;
}

#app {
  --clr-text: rgba(74,74,74,1);
  --clr-accent: hsl(195, 81%, 41.2%);

  display: grid;
  place-content: center;
  font-family: "Open Sans", sans-serif;
  color: var(--clr-text);
  margin: 2rem auto;
}

.calculator {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.legal-input {
  display: grid;
  grid-template-columns: minmax(auto, 3fr) minmax(17ch, 2fr) 1ch;
  align-items: center;
  width: 100%;
  margin-bottom: 2rem;
}

.legal-input select {
  font-family: inherit;
  font-size: inherit;
  background: rgba(10, 36, 51, 0.1);
  border: none;
  border-radius: 10px;
  margin-right: 0.5rem;
  padding-right: 0.5rem;
  text-align: right;
  height: 2.2rem;
  appearance: none;

  /* no <select>-text-align in Safari :( */
  padding-left: 0.75rem;
}

.legal-input select:focus {
  outline: none;
  box-shadow: 0 0 0 1px var(--clr-accent);
}

.warning {
  color: tomato;
  font-weight: bold;
  margin-bottom: 1rem;
}

.calc-btn {
  padding: 1rem 2rem;
  font-family: inherit;
  font-size: inherit;
  color: #fff;
  background: var(--clr-accent);
  border: none;
  border-radius: 9999px;
  cursor: pointer;
}

.output-block {
  width: 100%;
  margin-top: 3rem;
}

</style>

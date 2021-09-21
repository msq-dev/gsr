<template>
  <div id="app">
    <div class="calculator">
      <div class="input-block">

        <label for="gewinn">Gewinn aus Gewerbebetrieb</label>
        <input
          ref="gewinn"
          id="gewinn"
          type="text"
          inputmode="decimal"
          v-model="gewinn"
          @keyup.enter="validate()"
          autofocus
        ><span>€</span>

        <label for="hinzurechnung">Hinzurech&shy;nungs&shy;betrag</label>
        <input
          ref="hinzurechnung"
          id="hinzurechnung"
          type="text"
          inputmode="decimal"
          v-model="hinzurechnung"
          @keyup.enter="validate()"
        ><span>€</span>

        <label for="kuerzung">Kürzungsbetrag</label>
        <input
          ref="kuerzung"
          id="kuerzung"
          type="text"
          inputmode="decimal"
          v-model="kuerzung"
          @keyup.enter="validate()"
        ><span>€</span>

        <label for="hebesatz">Hebesatz</label>
        <input
          ref="hebesatz"
          id="hebesatz"
          type="text"
          inputmode="decimal"
          v-model="hebesatz"
          @keyup.enter="validate()"
        ><span>%</span>

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
        <span>Gewinn aus Gewerbe</span>
        <span class="result">{{ germanize(cleanInput(gewinn)) }} €</span>

        <span>Hinzurech&shy;nungs&shy;betrag</span>
        <span class="result">+ {{ germanize(cleanInput(hinzurechnung)) }} €</span>

        <span>Kürzungsbetrag</span>
        <span class="result">&minus; {{ germanize(cleanInput(kuerzung)) }} €</span>

        <span class="last-calc">Freibetrag</span>
        <span class="result">&minus; {{ germanize(freibetrag) }} €</span>

        <span>Maßgebender Gewerbeertrag</span>
        <span class="result">{{ germanize(gewerbeErtrag) }} €</span>

        <span class="last-calc">Gewerbest.-Messzahl</span>
        <span class="result">&times; 3,50 %</span>

        <span>Steuer&shy;mess&shy;betrag</span>
        <span class="result">{{ germanize(steuerMessBetrag) }} €</span>

        <span>Hebesatz</span>
        <span class="result">&times; {{ germanize(cleanInput(hebesatz)) }} %</span>

        <span class="final-result">Gewerbe&shy;steuer</span>
        <span class="result">{{ germanize(gewerbeSteuer) }} €</span>
      </div>

    </div>
  </div>

</template>

<script>

export default {
  name: 'App',
  components: {
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
        { value: this.gewinn, inputField: this.$refs.gewinn },
        { value: this.hinzurechnung, inputField: this.$refs.hinzurechnung },
        { value: this.kuerzung, inputField: this.$refs.kuerzung },
        { value: this.hebesatz, inputField: this.$refs.hebesatz }
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
        this.$refs.gewinn.classList.add("invalid")
        this.profitMissing = true
      }

      if (!this.invalidInput && !this.profitMissing) {
        this.calculate()
      }

    },
    calculate() {
      const freiBetraege = {
        "Einzelunternehmen": 24500,
        "Personengesellschaft": 24500,
        "Kapitalgesellschaft": 0,
        "rechtsfähiger Verein": 5000
      }
      
      this.freibetrag = freiBetraege[this.rechtsform] ? freiBetraege[this.rechtsform] : 0
      this.gewerbeErtrag = this.cleanInput(this.gewinn) + this.cleanInput(this.hinzurechnung) - this.cleanInput(this.kuerzung) - this.freibetrag
      this.steuerMessBetrag = Math.floor(this.gewerbeErtrag * 0.035)
      this.hebesatz = this.hebesatz ? this.hebesatz : "200"
      this.gewerbeSteuer = this.steuerMessBetrag * (this.cleanInput(this.hebesatz) / 100)

      this.calculated = true
    },

    cleanInput(input) {
      input = input ? input : "0"
      input = input.replaceAll(" ", "")
      input = input.replaceAll(",", ".")
      return parseFloat(input)
    },

    germanize(number) {
      return number.toFixed(2).replace(".", ",")      
    }
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

.input-block,
.output-block {
  display: grid;
  grid-template-columns: minmax(auto, 3fr) minmax(17ch, 2fr) 1ch;
  row-gap: 0.75rem;
  margin-bottom: 1rem;
  width: 100%;
}

.input-block {
  align-items: center;
}

.input-block label {
  margin-right: 0.5rem;
}

.input-block input,
.input-block select {
  font-family: inherit;
  font-size: inherit;
  background: rgba(10, 36, 51, 0.1);
  border: none;
  border-radius: 10px;
  margin-right: 0.5rem;
  padding-right: 0.5rem;
  text-align: right;
  height: 2.2rem;
}

/* no <select>-text-align in Safari :( */
.input-block select {
  padding-left: 0.75rem;
}

.input-block select {
  appearance: none;
}

.input-block input:focus,
.input-block select:focus {
  outline: none;
  box-shadow: 0 0 0 1px var(--clr-accent);
}

.input-block span {
  font-size: inherit;
}

input.invalid {
  box-shadow: 0 0 0 2px tomato !important;
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
  grid-template-columns: minmax(auto, 3fr) minmax(20ch, 2fr);
  margin-top: 3rem;
  align-items: bottom;
}

.result {
  font-family: "Azeret Mono", monospace;
  text-align: right;
}

.last-calc,
.last-calc + span {
  padding-bottom: 0.25rem;
  border-bottom: 1px solid #dedede;
  margin-bottom: 0.25rem;
}

.final-result,
.final-result + span {
  font-size: 110%;
  font-weight: bold;
}
</style>

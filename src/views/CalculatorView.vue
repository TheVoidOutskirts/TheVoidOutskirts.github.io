<template>
  <div class="container">
    <div class="row">
      <!-- Selezione attaccante -->
      <div>
        <div class="col-6">
          <div class="mb-4">
            <h2 class="text-center">Scegli un attaccante</h2>
            <!--suppress HtmlUnknownTag -->
            <v-select
                :options="Personaggi"
                label="nome"
                v-model="attacker"></v-select>
          </div>
          <div v-show="attacker" class="mb-4">
            <h2>Dettagli personaggio</h2>
            <div><span class="h4">Nome:</span> <span class="ps-3">{{ attacker?.nome }}</span></div>
            <div><span class="h4">Armatura:</span> <span class="ps-3">{{ attacker?.armatura }}</span></div>
            <div>
              <span class="h4">Competenza Attacco:</span> <span class="ps-3">{{ attacker?.competenzaAttacco }}</span>
            </div>
            <div>
              <span class="h4">Competenza Difesa:</span> <span class="ps-3">{{ attacker?.competenzaDifesa }}</span>
            </div>
            <div>
              <span class="h4">Resistenza al danno:</span>
              <span class="ps-3">{{ attacker?.resistenzaAlDanno ?? 0 }}</span>
            </div>
            <div>
              <span class="h4">Armi:</span>
              <div v-if="attackerWeapons.length === 0">Il personaggio non ha armi</div>
              <div v-else>
                <v-select
                    :options="attackerWeapons"
                    label="nome"
                    v-model="weapon"></v-select>
              </div>
              <div class="form-check">
                <input class="form-check-input" type="checkbox" value="" id="attackerAllWeaponsCheckbox"
                       v-model="useAllWeapons">
                <label class="form-check-label" for="attackerAllWeaponsCheckbox">Mostra tutte le armi</label>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Probabilità dell'arma -->
      <div class="row justify-content-center">
        <div class="col-sm-10 col-md-10 col-lg-8 col-xl-8 col-xxl-8">
          <BarChart :data="weaponPercentageChartData"/>
        </div>
      </div>
    </div>

    <hr>

    <!-- Selezione difensore -->
    <div class="mt-4">
      <div class="row">
        <div class="col-6">
          <div class="mb-4">
            <h2 class="text-center">Scegli un difensore</h2>
            <v-select :options="Personaggi"
                      label="nome"
                      v-model="defender"></v-select>
          </div>
          <div v-show="defender">
            <h2>Dettagli personaggio</h2>
            <div><span class="h4">Nome:</span> <span class="ps-3">{{ defender?.nome }}</span></div>
            <div><span class="h4">Armatura:</span> <span class="ps-3">{{ defender?.armatura }}</span></div>
            <div>
              <span class="h4">Competenza Attacco:</span> <span class="ps-3">{{ defender?.competenzaAttacco }}</span>
            </div>
            <div>
              <span class="h4">Competenza Difesa:</span> <span class="ps-3">{{ defender?.competenzaDifesa }}</span>
            </div>
            <div>
              <span class="h4">Resistenza al danno:</span>
              <span class="ps-3">{{ defender?.resistenzaAlDanno ?? 0 }}</span>
            </div>
            <div>
              <span class="h4">Copertura:</span>
              <div>
                <div class="form-check" v-for="(cover, index) in ValoriCopertura" :key="index">
                  <input class="form-check-input" type="radio" :value="cover" v-model="coverValue">
                  <label class="form-check-label">{{
                      // todo change cover type to object with string
                      index === "pesante" ? "Copertura pesante" :
                          index === "nessuna" ? "Nessuna copertura" :
                              index === "leggera" ? "Leggera copertura" :
                                  index === "alta" ? "Alta esposizione" : ""
                    }}</label>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Grafico probabilità attacco -->
      <div class="row justify-content-center">
        <div class="col-sm-10 col-md-10 col-lg-8 col-xl-8 col-xxl-8">
          <BarChart :data="attackPercentageChartData"/>
        </div>
      </div>
    </div>

    <div class="mt-4" v-show="tabellaProbabilita.length > 0">
      <hr/>
      <!-- Danno Medio -->
      <!--<h2 class="text-center">Danno Medio: <strong>{{ dannoMedio }} - {{ dannoMedio2 }}</strong></h2>-->
      <h2 class="text-center">Danno Medio: <strong>{{ parseFloat(dannoMedio.toFixed(2)) }}</strong></h2>
      <!-- Tabella probabilità danni -->
      <table class="table table-dark table-striped mt-2" id="tabDanni">
        <thead>
        <tr>
          <th>Risultato dadi</th>
          <th>Danno effettivo</th>
          <th>Probabilità</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="(row, index) in tabellaProbabilita" :key="index">
          <td>{{ parseFloat(row.probability.diceResult.toFixed(2)) }}</td>
          <td>{{ parseFloat(row.effectiveDamage.toFixed(2)) }}</td>
          <td>{{ parseFloat(row.probability.probability.toFixed(2)) }} %</td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup lang="ts">
import {ValoriCopertura} from "@/assets/Copertura";

import {computed, onMounted, ref, watch} from "vue";
import BarChart from "@/components/BarChart.vue";

import type {Arma, Armatura, Personaggio} from "@/assets/Types";

import type {ChartDataset} from "chart.js";
import {useDataStore} from "@/stores/data";

const coverValue = ref<number | undefined>(undefined);

const attacker = ref<Personaggio | null>(null);
const defender = ref<Personaggio | null>(null);

const useAllWeapons = ref(false);

const store = useDataStore();

// Fix for loading because I don't know how to get a ref from a store getter... todo find better way
watch(() => store.isDataLoaded, loadData)
// Mounted needed to load data for the first time
onMounted(loadData)

function loadData() {
  Armi.value = store.getArmi;
  Personaggi.value = store.getPersonaggi;
  Armature.value = store.getArmature;
}

const Armi = ref<Arma[]>([]);
const Personaggi = ref<Personaggio[]>([]);
const Armature = ref<Armatura[]>([]);

const attackerWeapons = computed<Arma[]>(() => {
  if (attacker.value == null) return [];
  // todo return not only all weapons, but also the weapons owned by the character (by marking their name in some way)
  // this to be able to use modifications owned by the player!
  if (useAllWeapons.value) return Armi.value;

  const weapons: Arma[] = [];

  attacker.value.armi.forEach(wc => {
    const foundWeapon = Armi.value.find(w => w.codice == wc.arma)
    if (foundWeapon !== undefined) {
      weapons.push(foundWeapon);
    }
  })

  return weapons;
});

const weapon = ref<Arma | null>(null);

// Fix for vue-select not emitting an event when the selection is cleared.
watch(attacker, () => {
  weapon.value = attackerWeapons.value[0];
})

function calcolaPercentualeAttacco(arma: Arma, attaccante: Personaggio, difensore: Personaggio, copertura: number): number[] {
  const probVett = [0.7000, 0.7375, 0.7750, 0.8125, 0.8500, 0.8875, 0.9250, 0.9625]

  const armaturaDifensore: Armatura | undefined = Armature.value.find(e => e.codice == difensore.armatura)
  if (!armaturaDifensore)
    throw `Armatura difensore non trovata ${difensore.armatura}`

  const tipoDanno = arma.tipoDanno;
  const gravitaDanno = arma.gravitaDanno
  const resistenza = armaturaDifensore.gravitaDanno?.[gravitaDanno]?.[tipoDanno][0] ?? 0;

  const probArray: number[] = new Array(30)
  for (let i = 0; i < 30; i++) {
    if (i < 8 && arma.tipoDanno == 'perforante')
      probArray[i] = Math.round(arma.probabilita[i] * (100 + 5 * (attaccante.competenzaAttacco - difensore.competenzaDifesa) - resistenza * probVett[i] - copertura) / 100)
    else
      probArray[i] = Math.round(arma.probabilita[i] * (100 + 5 * (attaccante.competenzaAttacco - difensore.competenzaDifesa) - resistenza - copertura) / 100)
  }
  return probArray.map(clamp)
}

const clamp = (x: number) => Math.min(99, Math.max(1, x))

const weaponPercentage = computed(() => {
  return weapon.value?.probabilita
})

const weaponPercentageChartData = computed<ChartDataset<'bar', number[]>[]>(() => {
  return [{
    label: "Probabilità di colpire base dell'arma",
    data: weaponPercentage.value ?? [],
    backgroundColor: ['rgba(54, 162, 235, 0.8)'],
    borderColor: ['rgba(54, 162, 235, 1)'],
    borderWidth: 1
  },]
})

const attackPercentage = computed(() => {
  if (weapon.value === null) return undefined;
  if (attacker.value === null) return undefined;
  if (defender.value === null) return undefined;
  if (coverValue.value === undefined) return undefined;
  return calcolaPercentualeAttacco(weapon.value, attacker.value, defender.value, coverValue.value);
})

interface ResultProbability {
  diceResult: number;
  probability: number;
}

interface DiceRow {
  probability: ResultProbability
  effectiveDamage: number;
}

const tabellaProbabilita = computed<DiceRow[]>(() => {
  if (weapon.value === null) return [];
  if (attacker.value === null) return [];
  if (defender.value === null) return [];
  if (coverValue.value === undefined) return [];

  const nomeArmaturaDifensore = defender.value.armatura
  const armaturaDifensore = Armature.value.find(e => e.codice == nomeArmaturaDifensore)
  const dannoMax = weapon.value.danno.reduce((a, b) => a + b, 0)
  const dannoMin = weapon.value.danno.length
  const totComb = weapon.value.danno.reduce((a, b) => a * b, 1)
  const tipoAttacco = weapon.value.tipoDanno
  const gravitaDanno = weapon.value.gravitaDanno
  const resistenza = armaturaDifensore?.gravitaDanno?.[gravitaDanno]?.[tipoAttacco][0] ?? 0

  const sum = (a: number[], b: number[]) => a.length > b.length
      ? a.map((_, i) => i < b.length ? a[i] + b[i] : a[i])
      : b.map((_, i) => i < a.length ? a[i] + b[i] : b[i])

  const dicePossibilities = (dices: number[]) => dices.reduce((_: number[], face) => _.reduceRight((s: number[], x) => [0, ...sum(Array(face).fill(x), s)], []), [1])

  const diceProbabilities = dicePossibilities(weapon.value.danno)
      .map(x => x / totComb * 100)
      .map<ResultProbability>((a, i) => ({diceResult: i, probability: a}))
      .filter(p => p.probability > 0.0)

  return diceProbabilities.map<DiceRow>((p, i) => {
    const damage = Math.max(0, (i + dannoMin) * (100 - resistenza) / 100 - (defender.value?.statistiche?.ossatura?.[1] ?? 0))
    return {probability: p, effectiveDamage: damage}
  });
});

const dannoMedio = computed<number>(() => {
  return tabellaProbabilita.value
      .map(row => row.effectiveDamage * row.probability.probability / 100)
      .reduce((a, b) => a + b, 0)
});

/*const dannoMedio2 = computed<number>(() => {
  return tabellaProbabilita.value.reduce((a, b) => b.probability.probability > a.probability.probability ? b : a, {
    effectiveDamage: 0,
    probability: {probability: 0, diceResult: 0}
  }).effectiveDamage
});*/

const attackPercentageChartData = computed<ChartDataset<'bar', number[]>[]>(() => {
  return [{
    label: "Probabilità di colpire",
    data: attackPercentage.value ?? [],
    backgroundColor: ['rgba(54, 162, 235, 0.8)'],
    borderColor: ['rgba(54, 162, 235, 1)'],
    borderWidth: 1
  }]
})
</script>

<style scoped>
/*https://vue-select.org/guide/css.html#dark-mode-example*/
>>> {
  --vs-controls-color: #664CC3;
  --vs-border-color: #664CC3;

  --vs-dropdown-bg: #282C34;
  --vs-dropdown-color: #CC99CD;
  --vs-dropdown-option-color: #CC99CD;

  --vs-selected-bg: #664CC3;
  --vs-selected-color: #EEEEEE;

  --vs-search-input-color: #EEEEEE;

  --vs-dropdown-option--active-bg: #664CC3;
  --vs-dropdown-option--active-color: #EEEEEE;
}
</style>

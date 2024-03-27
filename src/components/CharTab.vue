<template>
  <div class="scrollable-tab">
    <div class="q-px-sm q-mt-sm justify-between">
      <q-input class="row" label="Name" v-model="app.char.name" dense />

      <div class="row q-gutter-sm">
        <q-select
          class="col"
          options-selected-class="text-purple-2"
          label="Age"
          v-model="app.char.age"
          :options="Object.values(EAge)"
          dense
        />
        <q-input class="col" label="Movement" type="number" v-model.number="app.char.movement" dense />
      </div>

      <div class="row q-gutter-sm">
        <q-input class="col" label="Kin" v-model="app.char.kin" dense />
        <q-input class="col" label="Profession" v-model="app.char.profession" dense />
      </div>

      <q-input class="row" label="Weakness" v-model="app.char.weakness" dense autogrow />

      <q-input class="row" label="Appearance" v-model="app.char.appearance" dense autogrow />
    </div>

    <div class="row q-ml-sm items-center">
      <div class="col text-h6 text-bold">Edit Attributes</div>
      <div class="q-px-none">
        <q-toggle v-model="editAttributes" icon="mdi-pencil" />
      </div>
    </div>

    <div v-if="editAttributes" class="row q-mt-sm justify-evenly q-gutter-sm">
      <q-btn v-if="statsRolled" class="col-12 q-mb-sm" icon="mdi-dice-d20" flat @click="rollStats" label="Roll stats">
        <q-tooltip>Roll stats</q-tooltip>
      </q-btn>
      <q-input class="col-3" type="number" :label="EAttr.STR" v-model="app.char.attributes.STR.score" />

      <q-input class="col-3" type="number" :label="EAttr.CON" v-model="app.char.attributes.CON.score" />

      <q-input class="col-3" type="number" :label="EAttr.AGL" v-model="app.char.attributes.AGL.score" />

      <q-input class="col-3" type="number" :label="EAttr.INT" v-model="app.char.attributes.INT.score" />

      <q-input class="col-3" type="number" :label="EAttr.WIL" v-model="app.char.attributes.WIL.score" />

      <q-input class="col-3" type="number" :label="EAttr.CHA" v-model="app.char.attributes.CHA.score" />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue';

import { EAge, EAttr } from 'src/components/models';

import { useCharacterStore } from 'src/stores/character';
import { useQuasar } from 'quasar';

export default defineComponent({
  name: 'CharTab',
  setup() {
    const app = useCharacterStore();
    const editAttributes = ref(false);
    const $q = useQuasar();
    const rollStats = () =>
      $q
        .dialog({
          message: 'Roll and apply Character Stats?',
          maximized: true,
          cancel: true,
        })
        .onOk(() => {
          const r = (): number => {
            let sum = 0;

            let rolls: number[] = new Array(4).fill(0);
            rolls.forEach((n, i) => (rolls[i] = Math.floor(Math.random() * 6) + 1));
            rolls.sort();
            rolls.shift();
            rolls.forEach((roll) => (sum += roll));

            return sum;
          };

          Object.keys(EAttr).forEach((attr) => (app.char.attributes[attr as EAttr].score = r()));

          const hp = app.char.attributes[EAttr.CON].score;
          app.char.hp.max = hp;
          app.char.hp.current = hp;

          const wp = app.char.attributes[EAttr.WIL].score;
          app.char.wp.max = wp;
          app.char.wp.current = wp;
        });
    const statsRolled = computed((): boolean => {
      let total = 0;
      Object.keys(EAttr).forEach((attr) => (total += app.char.attributes[attr as EAttr].score));
      return total == 0;
    });
    return {
      app,
      EAge,
      EAttr,
      editAttributes,
      rollStats,
      statsRolled,
    };
  },
});
</script>

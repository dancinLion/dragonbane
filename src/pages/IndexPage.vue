<template>
  <!-- file deepcode ignore PureFunctionReturnValueIgnored: The return value is passed to a component -->
  <q-page class="column" :padding="$q.screen.gt.sm">
    <div class="row q-mt-xs justify-evenly">
      <div class="col-4">
        <char-attr :label="EAttr.STR" v-model="app.char.attributes.STR" />
      </div>

      <div class="col-4">
        <char-attr :label="EAttr.CON" v-model="app.char.attributes.CON" />
      </div>

      <div class="col-4">
        <char-attr :label="EAttr.AGL" v-model="app.char.attributes.AGL" />
      </div>

      <div class="col-4">
        <char-attr :label="EAttr.INT" v-model="app.char.attributes.INT" />
      </div>

      <div class="col-4">
        <char-attr :label="EAttr.WIL" v-model="app.char.attributes.WIL" />
      </div>

      <div class="col-4">
        <char-attr :label="EAttr.CHA" v-model="app.char.attributes.CHA" />
      </div>
    </div>

    <div class="row q-pl-xs q-pr-sm q-mt-xs">
      <points-block v-model="app.char.hp" label="HP" />
    </div>
    <div class="row q-pl-xs q-pr-sm">
      <points-block v-model="app.char.wp" label="WP" />
    </div>

    <q-tabs v-model="tab" align="justify" dense>
      <q-tab class="q-px-none" name="skills" label="Skills" />
      <q-tab class="q-px-none" name="combat" label="Combat" />
      <q-tab class="q-px-none" name="abilities" label="Abilities" />
      <q-tab class="q-px-none" name="gear" label="Gear" />
      <q-tab class="q-px-none" name="char" label="Char" />
    </q-tabs>

    <!-- This should simply fill the remaining available vertical space: -->
    <q-tab-panels v-model="tab" swipeable>
      <!--SKILLS-->
      <q-tab-panel name="skills" class="q-pa-none">
        <skills-tab />
      </q-tab-panel>

      <!--COMBAT-->
      <q-tab-panel name="combat" class="q-pa-none">
        <combat-tab />
      </q-tab-panel>

      <!--ABILITIES & SPELLS-->
      <q-tab-panel name="abilities" class="q-pa-none">
        <abilities-tab />
      </q-tab-panel>

      <!--GEAR-->
      <q-tab-panel name="gear" class="q-pa-none">
        <gear-tab />
      </q-tab-panel>

      <!--CHAR-->
      <q-tab-panel name="char" class="q-pa-none">
        <char-tab />
      </q-tab-panel>
    </q-tab-panels>
  </q-page>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';

import { EAge, EAttr } from 'src/components/models';

import { useCharacterStore } from 'src/stores/character';

import { BaseChance, DmgBonus } from 'src/lib/defaults';

import CharAttr from 'src/components/CharAttr.vue';
import PointsBlock from 'src/components/PointsBlock.vue';
import SkillsTab from 'src/components/SkillsTab.vue';
import CombatTab from 'src/components/CombatTab.vue';
import AbilitiesTab from 'src/components/AbilitiesTab.vue';
import GearTab from 'src/components/GearTab.vue';
import CharTab from 'src/components/CharTab.vue';

export default defineComponent({
  name: 'IndexPage',
  components: {
    CharAttr,
    PointsBlock,
    SkillsTab,
    CombatTab,
    AbilitiesTab,
    GearTab,
    CharTab,
  },
  setup() {
    const app = useCharacterStore();
    const tab = ref('skills');

    return {
      app,
      tab,
      EAttr,
      EAge,
      DmgBonus,
      BaseChance,
    };
  },
});
</script>

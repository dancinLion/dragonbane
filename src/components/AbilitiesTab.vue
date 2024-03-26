<template>
  <div class="scrollable-tab">
    <div class="row q-ml-sm items-center">
      <div class="col text-h6 text-bold">Heroic Abilities</div>
      <div class="q-px-none">
        <q-toggle v-model="editAbilities" icon="mdi-pencil" />
      </div>
    </div>

    <div v-if="app.conf.showSpells">
      <ability-block
        v-for="(ab, i) in app.char.abilities"
        :key="`abl-${i}`"
        v-model="app.char.abilities[i]"
        :edit-abilities="editAbilities"
        @delete="removeAbl(i)"
      />
      <q-btn
        v-if="editAbilities"
        class="q-ml-md q-mb-md"
        icon="add_circle"
        label="Add New Ability"
        dense
        @click="addAbl"
      />

      <div class="row q-ml-sm items-center">
        <div class="col text-h6 text-bold">Spells</div>
        <div class="q-px-none">
          <q-btn class="col-shrink" icon="sort" flat dense rounded @click="sortSpells">
            <q-tooltip>Sort spells by rank</q-tooltip>
          </q-btn>

          <q-checkbox
            class="col-shrink"
            v-model="showPreparedSpells"
            :edit-spells="editSpells"
            checked-icon="mdi-eye"
            unchecked-icon="mdi-eye-off"
            color="white"
          >
            <q-tooltip>Toggle prepared spells</q-tooltip>
          </q-checkbox>
          <q-toggle v-model="editSpells" icon="mdi-pencil" />
        </div>
      </div>

      <div class="row items-center q-ml-xs">
        <div class="col-shrink"><q-icon name="bookmark" size="sm" /></div>
        <div class="col-1">{{ spellsPrepared }}/{{ BaseChance(app.char.attributes.INT.score) }}</div>
        <div class="col-shrink">
          <q-icon name="mdi-book-variant" size="sm"></q-icon>
        </div>
        <div class="col-shrink" v-for="(r, i) in spellsByRank" :key="`ranked-spells-${i}`">
          <span class="q-pa-xs rounded-borders" v-if="r > 0">{{ i < 1 ? 'Tricks' : 'Rank ' + i }}: {{ r }}</span>
        </div>
      </div>

      <div v-for="(sp, i) in app.char.spells" :key="`spell-${i}`">
        <spell-block v-if="show(sp)" :edit-spells="editSpells" v-model="app.char.spells[i]" @delete="removeSpell(i)" />
      </div>
      <q-btn
        v-if="editSpells"
        class="q-ml-md q-mb-md"
        icon="add_circle"
        label="Add New Spell"
        dense
        @click="addSpell"
      />
    </div>
    <div v-else>
      <ability-block
        v-for="(ab, i) in app.char.abilities"
        :key="`abl-${i}`"
        v-model="app.char.abilities[i]"
        :edit-abilities="editAbilities"
        @delete="removeAbl(i)"
      />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, computed, ref } from 'vue';

import { useQuasar } from 'quasar';
import { useCharacterStore } from 'src/stores/character';

import { NewAbility, NewSpell, BaseChance } from 'src/lib/defaults';

import AbilityBlock from './AbilityBlock.vue';
import SpellBlock from './SpellBlock.vue';
import { ISpell } from './models';

export default defineComponent({
  name: 'AbilitiesTab',
  components: { AbilityBlock, SpellBlock },
  setup() {
    const app = useCharacterStore();

    const $q = useQuasar();
    const editAbilities = ref(false);
    const editSpells = ref(false);
    const addSpell = () => app.char.spells.push(NewSpell());
    const removeSpell = (index: number) =>
      $q
        .dialog({
          message: 'Delete this spell?',
          cancel: true,
          maximized: true,
        })
        .onOk(() => app.char.spells.splice(index, 1));
    const sortSpells = () =>
      app.char.spells.sort((a, b) => {
        if (a.rank < b.rank) return -1;
        if (a.rank > b.rank) return 1;
        else return 0;
      });
    const spellsByRank = computed((): number[] => {
      const spells = [0, 0, 0, 0, 0, 0];
      app.char.spells.forEach((sp) => {
        spells[sp.rank]++;
      });
      return spells;
    });
    const spellsPrepared = computed((): number => {
      let t = 0;
      app.char.spells.forEach((sp) => {
        if (sp.rank > 0 && sp.prepared) t++;
      });
      return t;
    });

    const addAbl = () => app.char.abilities.push(NewAbility());
    const removeAbl = (index: number) =>
      $q
        .dialog({
          message: 'Delete this ability?',
          cancel: true,
          maximized: true,
        })
        .onOk(() => app.char.abilities.splice(index, 1));

    const filter = ref('');
    const show = (s: ISpell): boolean => {
      if (s.rank > 0 && !s.prepared && showPreparedSpells.value) return false;
      if (filter.value == null || filter.value == '') return true;

      if (RegExp(filter.value, 'i').test(s.name) || RegExp(filter.value).test(s.text)) return true;

      return false;
    };

    const showPreparedSpells = ref(false);

    return {
      app,
      BaseChance,
      filter,
      show,
      editAbilities,
      editSpells,
      addSpell,
      removeSpell,
      sortSpells,
      spellsByRank,
      spellsPrepared,
      showPreparedSpells,

      addAbl,
      removeAbl,
    };
  },
});
</script>

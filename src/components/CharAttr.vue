<template>
  <div class="column items-center justify-center q-mt-xs">
    <!-- <q-btn @click="showRoller = true" flat rounded dense size="md"> -->
    <!-- <div class="text-bold">
        {{ label }}
        <q-icon name="mdi-dice-d20" size="sm"></q-icon>
      </div> -->
    <!-- </q-btn> -->

    <!-- <q-btn
        :class="`col-shrink text-bold q-pa-none ${attr.condition.check ? 'text-negative' : ''}`"
        size="xl"
        @click="showRoller = true"
        flat
        rounded
        > -->

    <!-- <div :class="`row text-bold text-h5 ${attr.condition.check ? 'text-negative' : ''}`"> -->

    <q-btn @click="showRoller = true" flat dense size="xs" class="q-px-sm">
      <div class="row items-center text-bold text-h5">
        <div class="text-overline q-mr-xs">{{ label }}</div>
        <div class="q-mr-xs">{{ attr.score }}</div>
        <q-icon name="mdi-dice-d20" size="sm" />
        <!-- <q-tooltip>Roll {{ label }} Check</q-tooltip> -->
      </div>
    </q-btn>

    <div class="text-bold"></div>

    <q-chip
      v-if="attr.condition.check"
      color="negative"
      class="q-pr-sm"
      dense
      clickable
      @click="attr.condition.check = false"
    >
      <q-icon name="mdi-skull" class="q-mr-xs" />
      {{ attr.condition.name }}
      <q-tooltip>Remove Condition</q-tooltip>
    </q-chip>

    <q-chip v-if="!attr.condition.check" color="transparent" dense clickable @click="attr.condition.check = true">
      <q-icon name="mdi-skull" color="dark" />
      <q-tooltip>Add Condition</q-tooltip>
    </q-chip>

    <!-- <q-checkbox
      v-if="!attr.condition.check"
      v-model="attr.condition.check"
      size="sm"
      left-label
      dense
      unchecked-icon="mdi-emoticon-happy"
      checked-icon="mdi-skull"
      color="white"
    /> -->
  </div>

  <q-dialog v-model="showRoller" maximized>
    <dice-roller
      :name="label"
      :banes="attr.condition.check ? 1 : 0"
      :target="attr.score"
      :roll-type="ERollType.Attr"
      @close="showRoller = false"
      @result="
        (r) =>
          notifySend(
            `${app.char.name} rolled ${label}: ${r}`,
            r.includes(ED20Result.Dragon) || r.includes(ED20Result.Success) ? 'SUCCESS' : 'ERROR'
          )
      "
    />
  </q-dialog>
</template>

<script lang="ts">
import { defineComponent, PropType, ref, watch } from 'vue';

import { ED20Result, ERollType, IAttribute } from './models';

import { useQuasar } from 'quasar';

import DiceRoller from './DiceRoller.vue';
import { useCharacterStore } from 'src/stores/character';
import { notifySend } from 'src/lib/notify';

export default defineComponent({
  name: 'CharStat',
  components: { DiceRoller },
  props: {
    modelValue: {
      type: Object as PropType<IAttribute>,
      required: true,
    },
    label: {
      type: String,
      required: true,
    },
  },
  emits: ['update:modelValue'],
  setup(props, { emit }) {
    const attr = ref(props.modelValue);
    watch(
      () => props.modelValue,
      () => (attr.value = props.modelValue),
      { deep: true }
    );
    watch(
      () => attr.value,
      () => emit('update:modelValue', attr.value),
      { deep: true }
    );

    const $q = useQuasar();
    const editAttr = () =>
      $q
        .dialog({
          title: `Edit ${props.label}`,
          cancel: true,
          prompt: {
            type: 'number',
            model: `${attr.value.score}`,
            min: 3,
            max: 18,
          },
          maximized: true,
        })
        .onOk((n) => (attr.value.score = n));

    const showRoller = ref(false);
    const app = useCharacterStore();

    return {
      app,
      attr,
      editAttr,
      showRoller,
      ERollType,
      ED20Result,
      notifySend,
    };
  },
});
</script>

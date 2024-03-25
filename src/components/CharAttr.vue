<template>
  <div class="column items-center justify-center q-ma-xs q-pa-xs">
    <!-- <q-btn @click="showRoller = true" flat rounded dense size="md"> -->
    <div class="text-bold">
      {{ label }}
    </div>
    <!-- </q-btn> -->

    <q-btn
      :class="`col-shrink text-bold q-pa-none ${attr.condition.check ? 'text-negative' : ''}`"
      size="xl"
      @click="showRoller = true"
      flat
      rounded
    >
      <div class="text-bold text-h5">{{ attr.score }}</div>
      <q-icon name="mdi-dice-d20" size="sm"></q-icon>
    </q-btn>
    <q-checkbox
      :label="attr.condition.name"
      v-model="attr.condition.check"
      size="sm"
      left-label
      dense
      unchecked-icon="mdi-emoticon-happy"
      checked-icon="mdi-skull"
      color="white"
    />
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
          send(
            `${app.char.name} rolled ${label}: ${r}`,
            r == ED20Result.Dragon || r == ED20Result.Success ? 'SUCCESS' : 'ERROR'
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
import { send } from 'src/lib/notify';

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
      send,
    };
  },
});
</script>

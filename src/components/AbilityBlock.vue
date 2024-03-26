<template>
  <q-expansion-item hide-expand-icon v-if="!editAbilities" class="items-center q-ma-none" dense>
    <template v-slot:header>
      <q-item-section class="col-grow q-ml-none text-bold">
        {{ abl.name }}
      </q-item-section>
      <q-item-section
        v-if="abl.wp > 0"
        :class="`col-1 text-bold ${app.char.wp.current < abl.wp ? 'text-negative' : ''}`"
      >
        {{ abl.wp }}&nbsp;WP
      </q-item-section>
      <q-item-section v-if="abl.wp > 0" class="col-1">
        <q-btn icon="mdi-arrow-right-bold-circle" flat dense @click="activate" />
      </q-item-section>
    </template>
    <template v-slot:default>
      <div class="q-mx-md">
        {{ abl.text }}
      </div>
    </template>
  </q-expansion-item>
  <div v-if="editAbilities" class="column q-mx-md q-mb-md">
    <div class="row q-gutter-sm">
      <q-input class="col" label="Name" v-model="abl.name" dense />
      <q-input class="col-1" label="WP" v-model.number="abl.wp" type="number" dense />
      <q-btn class="bg-negative" icon="delete" flat dense @click="$emit('delete')" />
    </div>
    <q-input class="row" label="Text" v-model="abl.text" dense autogrow />
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType, ref, watch } from 'vue';

import { IAbility } from './models';

import { useCharacterStore } from 'src/stores/character';
import { useQuasar } from 'quasar';

export default defineComponent({
  name: 'AbilityBlock',
  props: {
    editAbilities: Boolean,
    modelValue: {
      type: Object as PropType<IAbility>,
      required: true,
    },
  },
  emits: ['update:modelValue', 'delete'],
  setup(props, { emit }) {
    const abl = ref(props.modelValue);
    watch(
      () => props.modelValue,
      () => (abl.value = props.modelValue)
    );
    watch(
      () => abl.value,
      () => emit('update:modelValue', abl.value)
    );

    const $q = useQuasar();
    const app = useCharacterStore();
    const activate = () =>
      $q
        .dialog({
          title: `Use ${abl.value.name}?`,
          message: abl.value.text,
          prompt: {
            label: 'Spend WP',
            model: `${abl.value.wp}`,
            type: 'number',
            hint: `current WP: ${app.char.wp.current}`,
            max: app.char.wp.current,
          },
          ok: true,
          cancel: true,
          maximized: true,
        })
        .onOk((wp) => {
          if (app.char.wp.current >= +wp) app.char.wp.current -= +wp;
        });

    return {
      app,
      abl,
      activate,
    };
  },
});
</script>

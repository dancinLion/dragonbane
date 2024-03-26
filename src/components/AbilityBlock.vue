<template>
  <div>
    <q-expansion-item hide-expand-icon v-if="!editAbilities" class="items-center q-ma-none">
      <template v-slot:header>
        <q-item-section class="col-10 q-ml-none text-bold">
          {{ abl.name }}
        </q-item-section>
        <q-item-section class="text-bold">{{ abl.wp }}&nbsp;WP</q-item-section>
        <q-item-section>
          <q-btn icon="mdi-arrow-right-bold-circle" class="q-pl-sm" flat dense @click="activate" />
        </q-item-section>
      </template>
      <template v-slot:default>
        <div class="q-mx-md">
          {{ abl.text }}
        </div>
      </template>
    </q-expansion-item>
  </div>

  <div class="column q-mx-md q-mb-md" v-if="editAbilities">
    <div class="row">
      <q-input class="col" label="Name" v-model="abl.name" dense />
      <q-input class="col-xs-2 col-sm-1" label="WP" v-model.number="abl.wp" type="number" dense />
      <q-btn icon="delete" flat dense @click="$emit('delete')" />
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
      abl,
      activate,
    };
  },
});
</script>

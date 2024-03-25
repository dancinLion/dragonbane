<template>
  <div @click="showEditDialog = !showEditDialog" class="col-1 q-mr-xs items-center text-center text-h5 text-bold">
    {{ points.current }}
  </div>
  <div class="col health-bar q-mb-sm">
    <div
      v-for="n in points.max + 1"
      :key="n"
      :class="[
        `blip`,
        {
          [`full-${label.toLowerCase()}`]: n - 1 <= points.current,
          [`empty-${label.toLowerCase()}`]: n - 1 > points.current,
          zero: n === 1,
        },
      ]"
      @click="setHealth(n - 1)"
    >
      {{ displayDifference(n) }}
    </div>
  </div>

  <q-dialog v-model="showEditDialog" maximized>
    <q-card>
      <q-card-section class="row text-h6 items-center justify-between">
        <div class="col">Adjust {{ label }}</div>
        <q-btn class="col-shrink" icon="close" flat dense rounded @click="showEditDialog = false" />
      </q-card-section>

      <q-card-section class="column">
        <q-input label="Current" type="number" v-model.number="points.current" :max="points.max" :min="0" />
        <q-input label="Max" type="number" v-model.number="points.max" :min="0" />
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script lang="ts">
import { defineComponent, PropType, ref, watch } from 'vue';
import { IPoints } from './models';

export default defineComponent({
  name: 'PointsBlock',
  props: {
    modelValue: {
      type: Object as PropType<IPoints>,
      required: true,
    },
    label: {
      type: String,
      required: true,
    },
    showMax: {
      type: Boolean,
    },
  },
  methods: {
    displayDifference(n: number) {
      const difference = n - 1 - this.points.current;
      if (difference > 0) {
        return '+' + difference;
      } else if (difference < 0) {
        return difference.toString();
      }
      return '';
    },
  },
  emits: ['update:modelValue'],
  setup(props, { emit }) {
    const points = ref(props.modelValue);

    // Watch for external updates to modelValue and update local state
    watch(
      () => props.modelValue,
      () => {
        points.value = props.modelValue;
      }
    );

    // Watch for updates to local state and emit event to update parent
    watch(points, () => {
      emit('update:modelValue', points.value);
    });

    // Method to update current health based on blip clicked
    const setHealth = (newHealth: number) => {
      points.value = { ...points.value, current: newHealth }; // Update the current health
      emit('update:modelValue', points.value); // Emit the update to the parent component
    };

    const showEditDialog = ref(false);

    return {
      points,
      setHealth,
      showEditDialog,
    };
  },
});
</script>

<style>
.health-bar {
  display: flex;
  width: 100%;
  background-color: #eee;
  height: 30px;
}

.blip {
  flex: 1;
  transition: background-color 0.5s ease;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  user-select: none;
  cursor: pointer;
  font-size: 0;
  overflow: visible;
}

.blip:hover {
  font-size: 0.85em;
  font-weight: bold;
}

/* HP Styles */
.blip.full-hp {
  background-color: #f04242;
}
.blip.full-hp:hover {
  background-color: #ff6666;
}
.blip.empty-hp {
  background-color: #943838;
}
.blip.empty-hp:hover {
  background-color: #ff6666;
}

/* WP Styles */
.blip.full-wp {
  background-color: #42b5f0;
}
.blip.full-wp:hover {
  background-color: #66ccff;
}
.blip.empty-wp {
  background-color: #387594;
}
.blip.empty-wp:hover {
  background-color: #66ccff;
}

.blip.zero {
  flex: none;
  width: 20px;
  background-color: #2e2e38;
}

.blip.zero:hover {
  background-color: #2e2e38;
}
</style>

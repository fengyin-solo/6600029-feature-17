<script setup lang="ts">
import { computed } from 'vue';
import { useDroneStore } from '../store/drone';
import type { Waypoint } from '../types';

const store = useDroneStore();

const actions: { value: Waypoint['action']; label: string; icon: string }[] = [
  { value: 'none', label: '无', icon: '○' },
  { value: 'hover', label: '悬停', icon: '⏸' },
  { value: 'photo', label: '拍照', icon: '📷' },
  { value: 'video', label: '录像', icon: '🎥' },
];

const selected = computed(() => store.selectedWaypoint);

const altitude = computed({
  get: () => selected.value?.altitude ?? 0,
  set: (v: number) => {
    if (selected.value && !Number.isNaN(v)) {
      store.updateWaypoint(selected.value.id, { altitude: v });
    }
  },
});

const speed = computed({
  get: () => selected.value?.speed ?? 0,
  set: (v: number) => {
    if (selected.value && !Number.isNaN(v)) {
      store.updateWaypoint(selected.value.id, { speed: v });
    }
  },
});

const action = computed({
  get: () => selected.value?.action ?? 'none',
  set: (v: Waypoint['action']) => {
    if (selected.value) {
      store.updateWaypoint(selected.value.id, { action: v });
    }
  },
});

function handleDelete() {
  if (!selected.value) return;
  const idx = store.selectedWaypointIndex;
  const id = selected.value.id;
  store.removeWaypoint(id);
  const next = store.waypoints[idx] ?? store.waypoints[idx - 1] ?? null;
  store.selectWaypoint(next ? next.id : null);
}
</script>

<template>
  <div class="bg-slate-800 rounded-lg p-3 flex flex-col gap-3">
    <div class="flex items-center justify-between">
      <h3 class="text-xs font-semibold text-slate-300">航点详情</h3>
      <span class="text-[10px] text-slate-500">{{ store.waypoints.length }} 个航点</span>
    </div>

    <!-- Waypoint selector chips -->
    <div
      v-if="store.waypoints.length"
      class="flex flex-wrap gap-1 max-h-20 overflow-y-auto pr-1"
    >
      <button
        v-for="(wp, idx) in store.waypoints"
        :key="wp.id"
        @click="store.selectWaypoint(wp.id)"
        :class="store.selectedWaypointId === wp.id
          ? 'bg-sky-600 text-white border-sky-400'
          : 'bg-slate-700 text-slate-300 border-slate-600 hover:bg-slate-600'"
        class="px-2 py-0.5 rounded text-[10px] font-medium border transition"
      >
        {{ idx + 1 }}
      </button>
    </div>
    <p v-else class="text-[11px] text-slate-500">
      暂无航点 — 点击地图右上「点击添加」按钮后在地图打点
    </p>

    <!-- Detail editor -->
    <div v-if="selected" class="space-y-3 border-t border-slate-700 pt-2">
      <div class="flex items-center justify-between">
        <span class="text-xs text-sky-400 font-bold">
          WP{{ store.selectedWaypointIndex + 1 }}
        </span>
        <span class="text-[10px] text-slate-500">
          {{ selected.lat.toFixed(5) }}, {{ selected.lng.toFixed(5) }}
        </span>
      </div>

      <!-- Altitude -->
      <div class="space-y-1">
        <div class="flex justify-between text-[11px]">
          <span class="text-slate-400">高度</span>
          <span class="text-slate-200">
            {{ selected.altitude }}
            <span class="text-slate-500">m</span>
          </span>
        </div>
        <div class="flex items-center gap-2">
          <input
            type="range"
            min="10"
            :max="store.droneConfig.maxAltitude"
            step="5"
            v-model.number="altitude"
            class="flex-1 accent-sky-500"
          />
          <input
            type="number"
            min="0"
            :max="store.droneConfig.maxAltitude"
            v-model.number="altitude"
            class="w-16 bg-slate-900 border border-slate-700 rounded px-1 py-0.5 text-[11px] text-slate-100 focus:outline-none focus:border-sky-500"
          />
        </div>
      </div>

      <!-- Speed -->
      <div class="space-y-1">
        <div class="flex justify-between text-[11px]">
          <span class="text-slate-400">速度</span>
          <span class="text-slate-200">
            {{ selected.speed }}
            <span class="text-slate-500">m/s</span>
          </span>
        </div>
        <div class="flex items-center gap-2">
          <input
            type="range"
            min="1"
            :max="store.droneConfig.maxSpeed"
            step="1"
            v-model.number="speed"
            class="flex-1 accent-sky-500"
          />
          <input
            type="number"
            min="1"
            :max="store.droneConfig.maxSpeed"
            v-model.number="speed"
            class="w-16 bg-slate-900 border border-slate-700 rounded px-1 py-0.5 text-[11px] text-slate-100 focus:outline-none focus:border-sky-500"
          />
        </div>
      </div>

      <!-- Action -->
      <div class="space-y-1">
        <div class="text-[11px] text-slate-400">动作</div>
        <div class="grid grid-cols-4 gap-1">
          <button
            v-for="a in actions"
            :key="a.value"
            @click="action = a.value"
            :class="selected.action === a.value
              ? 'bg-sky-600 text-white'
              : 'bg-slate-700 text-slate-400 hover:bg-slate-600'"
            class="py-1.5 rounded text-[10px] font-medium transition flex flex-col items-center gap-0.5"
          >
            <span class="text-sm leading-none">{{ a.icon }}</span>
            <span>{{ a.label }}</span>
          </button>
        </div>
      </div>

      <!-- Actions -->
      <div class="flex gap-2 pt-1">
        <button
          @click="handleDelete"
          class="flex-1 py-1.5 rounded text-[11px] font-medium bg-red-800 text-white hover:bg-red-700 transition"
        >
          删除航点
        </button>
        <button
          @click="store.selectWaypoint(null)"
          class="px-3 py-1.5 rounded text-[11px] font-medium bg-slate-700 text-slate-300 hover:bg-slate-600 transition"
        >
          取消选择
        </button>
      </div>
    </div>

    <p
      v-else-if="store.waypoints.length"
      class="text-[11px] text-slate-500 border-t border-slate-700 pt-2"
    >
      选择上方编号或点击地图航点进行编辑
    </p>
  </div>
</template>

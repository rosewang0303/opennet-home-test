<template>
  <section>
    <ClientOnly>
      <VideoCard :source="vQ6" />
    </ClientOnly>

    <section class="mt-6">
      <!-- Match List -->
      <div
        v-for="match in matches"
        :key="match.id"
        class="rounded-lg mb-12 text-center"
      >
        <h3 class="font-bold">{{ match.home }} vs {{ match.away }}</h3>
        <div class="flex gap-2 mt-2">
          <button
            v-for="(odd, key) in match.odds"
            :key="key"
            @click="toggleSelection(match.id, key)"
            class="flex-1 px-4 py-2 rounded-lg font-bold text-lg"
            :class="[
              'px-2 py-1 rounded',
              selected[match.id]?.includes(key)
                ? 'bg-green-400'
                : 'bg-gray-100',
            ]"
          >
            {{ labelMap[key] }}: {{ odd }}
          </button>
        </div>
      </div>
      <!-- Selected Odds -->
      <h2 class="text-lg font-bold mt-4">
        Selected Odds: {{ calculateOdds() }}
      </h2>
    </section>
  </section>
</template>

<script lang="ts" setup>
import vQ6 from "@/assets/q6.mov";

defineOptions({
  name: "Q6",
});

/**
 * Team A vs Team B
 * Home Odds: 1.25
 * Away Odds: 3.75
 * Draw Odds: 10.00
 *
 * Team C vs Team D
 * Home Odds: 1.50
 * Away Odds: 2.50
 * Draw Odds: 12.00
 *
 * Team E vs Team F
 * Home Odds: 1.75
 * Away Odds: 2.25
 * Draw Odds: 8.00
 *
 * 請根據以上資料完成下列事項
 * 1. UI 切版
 * 2. 選取賽事後顯示總 Odds 計算結果 (選取的 Odds 相乘)
 * 3. 需要考慮 "互斥" 的狀況, 當狀況發生, 總 Odds 會變成一個範圍 (min~max)
 *
 * ** 互斥表示同一隊伍選擇多個結果, 這些結果不會同時發生, 因此稱為互斥 **
 *
 * e.g.
 * 情境1,
 * User 選擇 Team A 贏, Team C 贏, Team E 贏
 * 總 Odds 為 1.25 * 1.50 * 1.75 = 3.28125
 *
 * 情境2,
 * User 選擇 Team A 贏, Team C 贏, Team F 贏
 * 總 Odds 為 1.25 * 12.00 * 2.25 = 33.75
 *
 * 情境3 (互斥)
 * User 選擇 Team A 贏, Team B 贏, Team E 贏
 * 總 Odds-1 為 TeamA (1.25) * TeamE (1.75) = 2.1875
 * 總 Odds-2 為 TeamB (3.75) * TeamE (1.75) = 6.5625
 */

/**
 * Match data
 */
const matches = [
  {
    id: 1,
    home: "Team A",
    away: "Team B",
    odds: { home: 1.25, draw: 10.0, away: 3.75 },
  },
  {
    id: 2,
    home: "Team C",
    away: "Team D",
    odds: { home: 1.5, draw: 12.0, away: 2.5 },
  },
  {
    id: 3,
    home: "Team E",
    away: "Team F",
    odds: { home: 1.75, draw: 8.0, away: 2.25 },
  },
];

const labelMap: Record<string, string> = {
  home: "Home",
  away: "Away",
  draw: "Draw",
};

/**
 * Selected odds for each match
 */
const selected = ref<Record<number, string[]>>({});

/**
 * Toggle selection for a match
 */
function toggleSelection(matchId: number, key: string) {
  if (!selected.value[matchId]) {
    selected.value[matchId] = [];
  }
  const idx = selected.value[matchId].indexOf(key);
  if (idx > -1) {
    selected.value[matchId].splice(idx, 1);
  } else {
    selected.value[matchId].push(key);
  }
}

/**
 * Calculate total odds
 * - If single selection per match → simple multiplication
 * - If multiple selections (mutually exclusive) → calculate all combinations → return min~max
 */
function calculateOdds() {
  const oddsList: number[][] = [];

  for (const match of matches) {
    const sel = selected.value[match.id] || [];
    if (sel.length === 0) continue;
    const matchOdds = sel.map(
      (key) => match.odds[key as keyof typeof match.odds]
    );
    oddsList.push(matchOdds);
  }

  if (oddsList.length === 0) return "No selection";

  // Cartesian product
  let products = [1];
  for (const arr of oddsList) {
    const newProducts: number[] = [];
    for (const p of products) {
      for (const o of arr) {
        newProducts.push(p * o);
      }
    }
    products = newProducts;
  }

  if (products.length === 1) {
    return products[0]!;
  } else {
    return `${Math.min(...products)} ~ ${Math.max(...products)}`;
  }
}
</script>

<style></style>

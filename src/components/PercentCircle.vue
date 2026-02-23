<script lang="ts">
import { computed, defineComponent, ref, toRefs } from 'vue';

const PI = 3.1459;

export default defineComponent({
    name: 'PercentCircle',
    props: {
        fraction: {
            type: Number,
            required: true,
        },
        total: {
            type: Number,
            required: true,
        },
        label: {
            type: String,
            default: '',
        },
        radius: {
            type: Number,
            required: true,
        },
        unit: {
            type: String,
            default: '',
        },
    },
    setup(props) {
        const offset = ref(5);
        const { fraction, total, radius, unit } = toRefs(props);
        const diameter = computed(() => radius.value * 2);
        const circumference = computed(() => 2 * PI * radius.value);
        const containerWidth = computed(() => diameter.value + 2 * offset.value);
        const formatResourcePercent = computed(() =>
            fraction.value && total.value
                ? ((fraction.value / total.value) * 100.0).toFixed(2)
                : '0',
        );
        const strokeColor = computed(() =>
            parseFloat(formatResourcePercent.value) >= 90 ? 'red' : 'white',
        );
        const fractionUnits = computed(
            () => `${fraction.value}${unit.value}/${total.value}${unit.value}`,
        );
        const available = computed(() => Math.abs(total.value - fraction.value).toFixed(3));
        const dashArray = computed(() => {
            if (Number.isNaN(formatResourcePercent.value)) {
                return '0';
            }
            const scaledPath =
        (parseFloat(formatResourcePercent.value) / 100) * circumference.value;
            return `${scaledPath}, ${circumference.value}`;
        });

        return {
            offset,
            diameter,
            circumference,
            containerWidth,
            formatResourcePercent,
            strokeColor,
            fractionUnits,
            available,
            dashArray,
        };
    },
});
</script>

<template>
<div class="chart-container">
    <svg
        class="circular-chart"
        :style="{ 'max-width': containerWidth }"
        :viewBox="`${-offset * 6} ${-offset / 2} ${containerWidth} ${containerWidth}`"
    >
        <!-- Gradient definition -->
        <defs>
            <linearGradient id="gradient-stroke" x1="0%" y1="0%" x2="100%" y2="100%">
                <stop offset="0%" stop-color="#00D4FF" stop-opacity="1" />
                <stop offset="50%" stop-color="#4FACFE" stop-opacity="1" />
                <stop offset="100%" stop-color="#A855F7" stop-opacity="1" />
            </linearGradient>
            <linearGradient id="gradient-warning" x1="0%" y1="0%" x2="100%" y2="100%">
                <stop offset="0%" stop-color="#FF6B6B" stop-opacity="1" />
                <stop offset="100%" stop-color="#EE5A5A" stop-opacity="1" />
            </linearGradient>
            <!-- Glow filter -->
            <filter id="glow" x="-50%" y="-50%" width="200%" height="200%">
                <feGaussianBlur stdDeviation="2" result="coloredBlur"/>
                <feMerge>
                    <feMergeNode in="coloredBlur"/>
                    <feMergeNode in="SourceGraphic"/>
                </feMerge>
            </filter>
        </defs>
        <path class="circle-bg" :d="`M18 2 a ${radius} ${radius} 0 0 1 0 88 a ${radius} ${radius} 0 0 1 0 ${-diameter}`"/>
        <path
            class="circle"
            :stroke-dasharray="dashArray"
            :d="`M18 2 a ${radius} ${radius} 0 0 1 0 88 a ${radius} ${radius} 0 0 1 0 ${-diameter}`"
            :stroke="Number(formatResourcePercent) > 80 ? 'url(#gradient-warning)' : 'url(#gradient-stroke)'"
            :style="{ 'stroke-opacity' : Number.isNaN(formatResourcePercent) ? 0 : 1 }"
            filter="url(#glow)"
        />
        <text class="text label" x="18" :y="radius - offset">{{ label }}</text>
        <text class="text percentage" x="20" :y="radius + 12">{{ formatResourcePercent }}%</text>
    </svg>
    <p class="text usage">USED: {{ fraction }} {{ unit }}</p>
    <p class="text usage">TOTAL: {{ total }} {{ unit }}</p>
    <p class="text usage">AVAILABLE: {{ available }} {{ unit }}</p>
</div>
</template>

<style lang="sass" scoped>
.chart-container
  display: inline-block
  margin: 0.75rem
  padding: 0.75rem 1rem
  background: rgba(255, 255, 255, 0.03)
  border-radius: 16px
  border: 1px solid rgba(255, 255, 255, 0.06)
  transition: all 0.25s ease

  &:hover
    background: rgba(255, 255, 255, 0.06)
    border-color: rgba(79, 172, 254, 0.2)
    transform: translateY(-2px)

.circular-chart
  margin: 0 6px
  width: 8rem

.circle-bg
  fill: none
  stroke: rgba(255, 255, 255, 0.15)
  stroke-width: 3
  stroke-opacity: 0.6

.circle
  fill: none
  stroke-width: 6
  stroke-linecap: round
  animation: progress 1s ease-out forwards

@keyframes progress
  0%
    stroke-dasharray: 0 500

.text
  text-anchor: middle
  dominant-baseline: middle
  fill: white
  font-size: 14px

.label
  font-weight: 700
  font-size: 12px
  letter-spacing: 0.05em
  fill: rgba(255, 255, 255, 0.9)

.percentage
  font-weight: 600
  font-size: 16px
  fill: white
  filter: drop-shadow(0 1px 2px rgba(0, 0, 0, 0.3))

.usage
  font-size: 10px
  margin: 0.25rem 0 0 0
  color: rgba(255, 255, 255, 0.6)
  letter-spacing: 0.02em
  font-weight: 500
</style>

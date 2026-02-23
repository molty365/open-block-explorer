<script lang="ts">
import { defineComponent, computed } from 'vue';
import { useChainStore } from 'src/stores/chain';

export default defineComponent({
    name: 'MapData',
    props: {
        mapVisible: {
            type: Boolean,
            required: true,
        },
    },
    setup() {
        const chainStore = useChainStore();
        const HeadBlockProducer = computed(
            (): string => chainStore.head_block_producer,
        );
        const HeadBlock = computed((): number => chainStore.head_block_num);
        const lastIrreversibleBlock = computed(
            (): number => chainStore.last_irreversible_block_num,
        );

        return {
            HeadBlock,
            HeadBlockProducer,
            lastIrreversibleBlock,
        };
    },
});
</script>

<template>
<div class="stats-container row full-width text-center justify-center" :class="{'stats-on-map' : mapVisible}">
    <div class="stat-card">
        <div class="stat-icon">
            <q-icon name="view_in_ar" size="24px" />
        </div>
        <div class="stat-content">
            <div class="stat-label">Head Block</div>
            <div class="stat-value">{{HeadBlock.toLocaleString()}}</div>
        </div>
    </div>
    <div class="stat-card">
        <div class="stat-icon producing">
            <q-icon name="precision_manufacturing" size="24px" />
        </div>
        <div class="stat-content">
            <div class="stat-label">Producing</div>
            <div class="stat-value producer">{{HeadBlockProducer}}</div>
        </div>
    </div>
    <div class="stat-card">
        <div class="stat-icon">
            <q-icon name="verified" size="24px" />
        </div>
        <div class="stat-content">
            <div class="stat-label">Irreversible Block</div>
            <div class="stat-value">{{lastIrreversibleBlock.toLocaleString()}}</div>
        </div>
    </div>
</div>
</template>

<style scoped lang="sass">
// Stats container
.stats-container
  padding: 1rem 0.5rem
  gap: 1rem
  flex-wrap: wrap

// Individual stat card - glassmorphism style
.stat-card
  display: flex
  align-items: center
  gap: 1rem
  padding: 1rem 1.5rem
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.08) 0%, rgba(255, 255, 255, 0.03) 100%)
  backdrop-filter: blur(12px)
  -webkit-backdrop-filter: blur(12px)
  border-radius: 16px
  border: 1px solid rgba(255, 255, 255, 0.08)
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1), inset 0 1px 0 rgba(255, 255, 255, 0.05)
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1)
  position: relative
  overflow: hidden
  min-width: 200px

  // Gradient top border accent
  &::before
    content: ''
    position: absolute
    top: 0
    left: 0
    right: 0
    height: 2px
    background: linear-gradient(90deg, #00D4FF 0%, #4FACFE 33%, #6366F1 66%, #A855F7 100%)
    opacity: 0.6
    transition: opacity 0.3s ease

  &:hover
    transform: translateY(-3px)
    box-shadow: 0 8px 24px rgba(79, 172, 254, 0.15), 0 4px 12px rgba(0, 0, 0, 0.1), inset 0 1px 0 rgba(255, 255, 255, 0.08)
    border-color: rgba(79, 172, 254, 0.2)
    background: linear-gradient(135deg, rgba(255, 255, 255, 0.1) 0%, rgba(255, 255, 255, 0.04) 100%)

    &::before
      opacity: 1

    .stat-icon
      transform: scale(1.1)
      background: linear-gradient(135deg, rgba(79, 172, 254, 0.25) 0%, rgba(99, 102, 241, 0.25) 100%)

// Stat icon container
.stat-icon
  display: flex
  align-items: center
  justify-content: center
  width: 48px
  height: 48px
  border-radius: 12px
  background: linear-gradient(135deg, rgba(79, 172, 254, 0.15) 0%, rgba(99, 102, 241, 0.15) 100%)
  color: #4FACFE
  transition: all 0.3s ease
  flex-shrink: 0

  &.producing
    background: linear-gradient(135deg, rgba(34, 197, 94, 0.15) 0%, rgba(16, 185, 129, 0.15) 100%)
    color: #22c55e
    animation: pulse-producing 2s ease-in-out infinite

@keyframes pulse-producing
  0%, 100%
    box-shadow: 0 0 0 0 rgba(34, 197, 94, 0)
  50%
    box-shadow: 0 0 0 6px rgba(34, 197, 94, 0.15)

// Stat content
.stat-content
  text-align: left

.stat-label
  font-weight: 500
  font-size: 0.6875rem
  letter-spacing: 0.1em
  text-transform: uppercase
  color: rgba(0, 0, 0, 0.45)
  margin-bottom: 0.25rem

.stat-value
  font-weight: 700
  font-size: 1.25rem
  letter-spacing: -0.02em
  background: linear-gradient(135deg, #1A1A2E 0%, #4FACFE 100%)
  -webkit-background-clip: text
  -webkit-text-fill-color: transparent
  background-clip: text
  line-height: 1.2

  &.producer
    font-size: 1.125rem
    letter-spacing: 0

// On map (dark background) styling
.stats-on-map
  .stat-card
    background: linear-gradient(135deg, rgba(10, 14, 39, 0.6) 0%, rgba(15, 20, 51, 0.6) 100%)
    border-color: rgba(255, 255, 255, 0.1)
    box-shadow: 0 4px 24px rgba(0, 0, 0, 0.3), inset 0 1px 0 rgba(255, 255, 255, 0.05)

    &:hover
      background: linear-gradient(135deg, rgba(10, 14, 39, 0.7) 0%, rgba(15, 20, 51, 0.7) 100%)
      border-color: rgba(79, 172, 254, 0.25)
      box-shadow: 0 8px 32px rgba(79, 172, 254, 0.2), 0 4px 16px rgba(0, 0, 0, 0.2)

  .stat-label
    color: rgba(255, 255, 255, 0.5)

  .stat-value
    background: linear-gradient(135deg, rgba(255,255,255,0.95) 0%, #4FACFE 100%)
    -webkit-background-clip: text
    -webkit-text-fill-color: transparent
    background-clip: text

  .stat-icon
    color: #4FACFE
    background: linear-gradient(135deg, rgba(79, 172, 254, 0.2) 0%, rgba(99, 102, 241, 0.2) 100%)

    &.producing
      color: #22c55e
      background: linear-gradient(135deg, rgba(34, 197, 94, 0.2) 0%, rgba(16, 185, 129, 0.2) 100%)

// Mobile responsive
@media screen and (max-width: 768px)
  .stats-container
    gap: 0.75rem
    padding: 0.75rem

  .stat-card
    min-width: 160px
    padding: 0.875rem 1rem
    gap: 0.75rem

  .stat-icon
    width: 40px
    height: 40px
    border-radius: 10px

    .q-icon
      font-size: 20px !important

  .stat-value
    font-size: 1.125rem

    &.producer
      font-size: 1rem

@media screen and (max-width: 480px)
  .stats-container
    flex-direction: column
    align-items: center

  .stat-card
    width: 100%
    max-width: 300px
    justify-content: flex-start

  .stat-content
    flex: 1
</style>

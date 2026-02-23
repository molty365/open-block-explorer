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
<div class="row full-width text-center justify-center actor-font" :class="{'text-grey-3' : mapVisible}">
    <div class="col-3">
        <div class="row">
            <div class="col-12 text-subtitle1 text-weight-thin text-uppercase">Head Block</div>
            <div class="col-12 text-subtitle1 text-bold">{{HeadBlock.toLocaleString()}}</div>
        </div>
    </div>
    <div class="col-1">
        <div class="hr-vertical"> </div>
    </div>
    <div class="col-3">
        <div class="row">
            <div class="col-12 text-subtitle1 text-weight-thin text-uppercase">Producing</div>
            <div class="col-12 text-subtitle1 text-bold">{{HeadBlockProducer}}</div>
        </div>
    </div>
    <div class="col-1">
        <div class="hr-vertical"></div>
    </div>
    <div class="col-3">
        <div class="row">
            <div class="col-12 text-subtitle1 text-weight-thin text-uppercase">Irreversible Block</div>
            <div class="col-12 text-subtitle1 text-bold">{{lastIrreversibleBlock.toLocaleString()}}</div>
        </div>
    </div>
</div>
</template>

<style scoped lang="sass">
.container
  margin-top: 9rem

hr
  content: ""
  display: block
  width: 50%
  border-size: 0.5rem
  border: 0px
  border-top: 2px solid var(--q-primary)
  margin-left: 1rem
  margin-right: 1rem

.hr-vertical
  content: ""
  display: block
  height: 100%
  border: 0px
  border-right: 2px solid rgba(79, 172, 254, 0.2)
  margin-left: 1rem
  margin-right: 1rem
  margin-top: -0.1rem
  position: relative

  &::after
    content: ''
    position: absolute
    top: 15%
    right: -1px
    width: 2px
    height: 70%
    background: linear-gradient(180deg, transparent 0%, rgba(79, 172, 254, 0.4) 50%, transparent 100%)
    border-radius: 2px

// Stats labels
.text-weight-thin
  font-weight: 500 !important
  font-size: 0.6875rem !important
  letter-spacing: 0.1em
  opacity: 0.55
  margin-bottom: 0.25rem

// Stats values
.text-bold
  font-weight: 700 !important
  font-size: 1.375rem !important
  letter-spacing: -0.02em
  background: linear-gradient(135deg, #4FACFE 0%, #6366F1 50%, #A855F7 100%)
  -webkit-background-clip: text
  -webkit-text-fill-color: transparent
  background-clip: text

// Stats container styling
.actor-font
  padding: 0.75rem 0
  position: relative

  &::before
    content: ''
    position: absolute
    top: 0
    left: 50%
    transform: translateX(-50%)
    width: 80%
    max-width: 600px
    height: 100%
    background: linear-gradient(180deg, rgba(79, 172, 254, 0.03) 0%, transparent 100%)
    border-radius: 12px
    pointer-events: none

// Grey text override for map visible state
.text-grey-3
  .text-bold
    background: linear-gradient(135deg, rgba(255,255,255,0.95) 0%, rgba(200,200,220,0.9) 100%)
    -webkit-background-clip: text
    -webkit-text-fill-color: transparent
    background-clip: text
  .text-weight-thin
    opacity: 0.7

@media screen and (max-width: 420px)
  .actor-font
    .text-bold
      font-size: 1.125rem !important
    .text-weight-thin
      font-size: 0.5625rem !important
      letter-spacing: 0.08em
</style>

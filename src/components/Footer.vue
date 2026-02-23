<script lang="ts">
import ConfigManager from 'src/config/ConfigManager';
import { computed } from 'vue';

export default {
    name: 'AppFooter',
    setup() {
        const footerLinks = computed(() => ConfigManager.get().getCurrentChain().getFooterLinks());

        return {
            footerLinks,
        };
    },
};
</script>

<template>

<div class="row footer-background justify-center text-center q-py-md">
    <div v-for="footerLink in footerLinks" :key="footerLink.label" class="col-lg-1 col-md-2 col-sm-2 col-xs-4 q-pa-md">
        <a class="no-dec" :href="footerLink.url" target="_blank">{{footerLink.label}}</a>
    </div>
</div>

</template>

<style lang="sass" scoped>
.router-link
    text-decoration: none
    color: #ffffff

.footer-background
    background: linear-gradient(180deg, #0A0E27 0%, #060912 100%)
    backdrop-filter: blur(14px)
    border-top: 1px solid rgba(79, 172, 254, 0.1)
    padding: 2rem 0 1.5rem
    position: relative
    overflow: hidden

    // Subtle gradient glow at top edge
    &::before
        content: ''
        position: absolute
        top: 0
        left: 50%
        transform: translateX(-50%)
        width: 60%
        max-width: 500px
        height: 1px
        background: linear-gradient(90deg, transparent 0%, rgba(79, 172, 254, 0.4) 25%, rgba(99, 102, 241, 0.5) 50%, rgba(168, 85, 247, 0.4) 75%, transparent 100%)

    // Ambient glow effect
    &::after
        content: ''
        position: absolute
        top: -50%
        left: 50%
        transform: translateX(-50%)
        width: 400px
        height: 200px
        background: radial-gradient(ellipse at center, rgba(79, 172, 254, 0.08) 0%, transparent 70%)
        pointer-events: none

.no-dec
    text-decoration: none
    color: rgba(255, 255, 255, 0.65)
    font-weight: 500
    font-size: 0.8125rem
    letter-spacing: 0.02em
    padding: 0.625rem 1.25rem
    border-radius: 10px
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1)
    display: inline-block
    position: relative
    overflow: hidden

    // Gradient border effect on hover
    &::before
        content: ''
        position: absolute
        inset: 0
        border-radius: 10px
        padding: 1px
        background: linear-gradient(135deg, rgba(79, 172, 254, 0.3) 0%, rgba(99, 102, 241, 0.3) 50%, rgba(168, 85, 247, 0.3) 100%)
        -webkit-mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0)
        -webkit-mask-composite: xor
        mask-composite: exclude
        opacity: 0
        transition: opacity 0.3s ease

    // Shine effect on hover
    &::after
        content: ''
        position: absolute
        top: 0
        left: -100%
        width: 100%
        height: 100%
        background: linear-gradient(90deg, transparent 0%, rgba(255, 255, 255, 0.1) 50%, transparent 100%)
        transition: left 0.5s ease

    &:hover
        color: #fff
        background: linear-gradient(135deg, rgba(79, 172, 254, 0.12) 0%, rgba(99, 102, 241, 0.12) 100%)
        transform: translateY(-2px)
        box-shadow: 0 4px 12px rgba(79, 172, 254, 0.2)

        &::before
            opacity: 1

        &::after
            left: 100%

    &:active
        transform: translateY(0)
        box-shadow: 0 2px 6px rgba(79, 172, 254, 0.15)

// Footer link container hover group effect
.q-pa-md
    transition: all 0.2s ease
    border-radius: 12px

    &:hover .no-dec
        color: rgba(255, 255, 255, 0.75)

// Copyright text styling (if added)
.footer-copyright
    color: rgba(255, 255, 255, 0.35)
    font-size: 0.75rem
    font-weight: 400
    letter-spacing: 0.02em
    margin-top: 1rem
    padding-top: 1rem
    border-top: 1px solid rgba(255, 255, 255, 0.04)
</style>

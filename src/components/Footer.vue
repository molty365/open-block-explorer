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
    background: linear-gradient(180deg, #0A0E27 0%, #080B1E 100%)
    backdrop-filter: blur(14px)
    border-top: 1px solid rgba(255, 255, 255, 0.06)
    padding: 1.5rem 0

.no-dec
    text-decoration: none
    color: rgba(255, 255, 255, 0.7)
    font-weight: 500
    font-size: 0.875rem
    letter-spacing: 0.01em
    padding: 0.5rem 1rem
    border-radius: 8px
    transition: all 0.2s ease
    display: inline-block

    &:hover
        color: #fff
        background: rgba(79, 172, 254, 0.1)
        transform: translateY(-1px)
</style>

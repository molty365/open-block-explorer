<script lang="ts">
import { defineComponent, computed, ref, watch } from 'vue';
import { useQuasar } from 'quasar';
import LoginHandler from 'components/LoginHandler.vue';
import HeaderSearch from 'components/HeaderSearch.vue';
import ChainsMenu from 'components/ChainsMenu.vue';
import ConfigManager, { getChain } from 'src/config/ConfigManager';
import { useRouteDataNetwork } from 'src/router';
import { HeaderSettings } from 'src/types/UiCustomization';
import { useAccountStore } from 'src/stores/account';

export default defineComponent({
    name: 'AppHeader',
    components: {
        LoginHandler,
        HeaderSearch,
        ChainsMenu,
    },
    setup() {
        const $q = useQuasar();
        const accountStore = useAccountStore();
        const headerSettings = computed((): HeaderSettings => ConfigManager.get().getCurrentChain().getUiCustomization().headerSettings);

        const account = computed(() => accountStore.accountName);
        const isLarge = computed((): boolean => $q.screen.gt.md);
        const showMultichainSelector = computed(() => process.env.SHOW_MULTICHAIN_SELECTOR === 'true');

        const isTestnet = ref(getChain().isTestnet());
        const smallLogoPath = ref(getChain().getSmallLogoPath());
        const largeLogoPath = ref(getChain().getLargeLogoPath());

        const network = useRouteDataNetwork();

        watch(network, () => {
            smallLogoPath.value = getChain().getSmallLogoPath();
            largeLogoPath.value = getChain().getLargeLogoPath();
            isTestnet.value = getChain().isTestnet();
        });

        return {
            headerSettings,
            account,
            isLarge: isLarge,
            showMultichainSelector,
            smallLogoPath,
            largeLogoPath,
            isTestnet,
        };
    },
});
</script>

<template>
<div class="header-background">
    <div class="row text-center q-pt-sm justify-between q-pt-md">
        <div class="logo-container col-xs-3 col-sm-2 col-md-2 col-lg-2">
            <div class="q-px-xs-xs q-px-sm-xs q-px-md-md q-px-lg-md">
                <div class="logo-header-container">
                    <div class="logo-chain-selector-container">
                        <a class="float-left" href="/">
                            <img v-if="isLarge" class="logo" :src="largeLogoPath">
                            <img v-else class="logo-token" :src="smallLogoPath">
                        </a>
                        <ChainsMenu v-if="showMultichainSelector"/>
                    </div>
                    <div v-if="isTestnet" class="testnet-text">TESTNET</div>
                </div>
            </div>
        </div>
        <div class="col-xs-4 col-sm-6 col-md-4 col-lg-6">
            <div class="q-px-xs-xs q-px-sm-xs q-px-md-md q-px-lg-md">
                <div class="row justify-center full-width">
                    <div class="col-12">
                        <HeaderSearch/>
                    </div>
                </div>
            </div>
        </div>
        <LoginHandler v-if="!headerSettings.hideLoginHandler"/>
    </div>
    <div class="row justify-center col-12 q-pt-sm">
        <q-tabs
            active-class="active-tab"
            indicator-color="white"
            align="justify"
            narrow-indicator
            color="white"
        >
            <q-route-tab
                v-if="!headerSettings.hideNetworkTab"
                class="deactive"
                name="network"
                label="Network"
                to="/network"
            />
            <q-route-tab
                v-if="!headerSettings.hideWalletTab && account"
                class="deactive"
                name="wallet"
                label="Wallet"
                :to="'/account/' + account + '/?tab=transactions'"
            />
            <q-route-tab
                v-if="!headerSettings.hideVoteTab"
                class="deactive"
                name="vote"
                label="Vote"
                to="/vote"
            />
            <q-route-tab
                v-if="!headerSettings.hideProposalTab"
                class="deactive"
                name="proposal"
                label="Proposal"
                to="/proposal"
            />
        </q-tabs>
    </div>
</div>
</template>

<style lang="sass" scoped>
.q-tab
    text-transform: unset
    font-size: 15px
    font-weight: 500
    letter-spacing: -0.01em
    transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1)
    border-radius: 10px
    margin: 0 0.25rem
    padding: 0.5rem 1.25rem
    min-height: 40px
    color: rgba(255, 255, 255, 0.65)
    position: relative

    &::before
        content: ''
        position: absolute
        inset: 0
        background: linear-gradient(135deg, rgba(79, 172, 254, 0.12) 0%, rgba(99, 102, 241, 0.12) 100%)
        opacity: 0
        transition: opacity 0.25s ease
        border-radius: 10px

    &:hover
        color: rgba(255, 255, 255, 0.9)
        &::before
            opacity: 1

    &.q-tab--active,
    &.active-tab
        color: #fff !important
        &::before
            opacity: 1
            background: linear-gradient(135deg, rgba(79, 172, 254, 0.2) 0%, rgba(99, 102, 241, 0.2) 100%)

.logo-header-container
    position: relative
    display: flex
    flex-direction: column
    gap: 8px
    width: fit-content

.logo-chain-selector-container
    padding-left: 4px
    display: flex
    flex-direction: row
    justify-content: space-between
    a
        margin-right: 4px

.logo
  width: 104px
  height: 40px
  object-fit: contain

.logo-token
  width: 40px
  height: 40px
  object-fit: contain

.testnet-text
    position: absolute
    bottom: -20px
    color: white
    font-size: 10px
    width: 100%
    height: min-content
    padding: 4px 12px
    border-radius: 50px
    background: linear-gradient(135deg, rgba(168, 85, 247, 0.3) 0%, rgba(99, 102, 241, 0.3) 100%)
    backdrop-filter: blur(8px)
    border: 1px solid rgba(255, 255, 255, 0.1)
    text-transform: uppercase
    letter-spacing: 0.05em
    font-weight: 600

.active-tab
  text-decoration: none
  color: #fff !important
  opacity: 1 !important

.deactive
  opacity: 1
  font-size: 15px

// Tab indicator glow effect
:deep(.q-tab__indicator)
  height: 3px !important
  border-radius: 3px 3px 0 0 !important
  background: linear-gradient(90deg, #00D4FF 0%, #4FACFE 33%, #6366F1 66%, #A855F7 100%) !important
  box-shadow: 0 0 12px rgba(79, 172, 254, 0.6), 0 0 24px rgba(99, 102, 241, 0.3)

.header-background
  background: rgba(10, 14, 39, 0.85)
  backdrop-filter: blur(20px)
  -webkit-backdrop-filter: blur(20px)
  border-bottom: 1px solid rgba(255, 255, 255, 0.06)
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.15)

</style>

<script lang="ts">
import { defineComponent, onMounted, ref, computed, watch } from 'vue';
import TransactionsTable from 'components/TransactionsTable.vue';
import TokensPanel from 'components/TokensPanel.vue';
import KeysPanel from 'components/KeysPanel.vue';
import ChildrenPanel from 'components/ChildrenPanel.vue';
import AccountCard from 'components/AccountCard.vue';
import ContractTabs from 'components/contract/ContractTabs.vue';
import BpVotes from 'components/BpVotes.vue';
import { api } from 'src/api';
import { useRoute, useRouter } from 'vue-router';
import { useAccountStore } from 'src/stores/account';
import ConfigManager from 'src/config/ConfigManager';

export default defineComponent({
    name: 'AccountPage',
    components: {
        TransactionsTable,
        TokensPanel,
        KeysPanel,
        ChildrenPanel,
        AccountCard,
        ContractTabs,
        BpVotes,
    },
    setup() {
        const accountStore = useAccountStore();
        const route = useRoute();
        const router = useRouter();
        const accountPageSettings = computed(() => ConfigManager.get().getCurrentChain().getUiCustomization().accountPageSettings);

        const tab = ref<string>((route.query['tab'] as string) || 'transactions');
        const account = computed(() => (route.params.account as string) || '');
        const abi = computed(() => accountStore.abi.abi);
        const tokenList = ref(api.getTokens(account.value));

        onMounted(async () => {
            await accountStore.updateABI(route.params.account as string);
            if (route.query.tab !== tab.value) {
                await updateQueryParams();
            }
        });

        watch([account], async () => {
            await accountStore.updateABI(route.params.account as string);
        });

        const updateQueryParams =  async () => {
            await router.replace({ query: { tab: tab.value } });
        };

        const onChangeTab = (newTab: string) => {
            tab.value = newTab;
        };

        watch([tab], () => {
            void router.push({
                query: {
                    tab: tab.value,
                },
            });
        });

        watch(route, () => {
            // handle tab update on browser navigation
            if (route.path.includes('/account/') && route.query.tab !== tab.value){
                onChangeTab(route.query.tab as string);
            }
        });

        return {
            onChangeTab,
            tab,
            account,
            abi,
            tokenList,
            accountPageSettings,
        };
    },
});
</script>

<template>
<div class="row col-12">
    <div class="column col-12 header-support">
        <div class="row">
            <AccountCard class="account-card" :account="account" :tokens="tokenList"/>
        </div>
        <q-tabs
            v-model="tab"
            class="account-view tabs"
            no-caps
            @update:model-value="onChangeTab"
        >
            <q-tab v-if="!accountPageSettings.hideTransactionTab" name="transactions" label="Transactions"/>
            <q-tab v-if="!accountPageSettings.hideContractsTab && abi" name="contract" label="Contract"/>
            <q-tab v-if="!accountPageSettings.hideTokensTab" name="tokens" label="Tokens"/>
            <q-tab v-if="!accountPageSettings.hideKeysTab" name="keys" label="Keys"/>
            <q-tab name="votes" label="Votes"/>
            <q-tab v-if="!accountPageSettings.hideChildrenTab" name="children" label="Children"/>
        </q-tabs>
    </div>
    <q-tab-panels v-model="tab" class="col-12">
        <q-tab-panel v-if="!accountPageSettings.hideTransactionTab" name="transactions">
            <TransactionsTable :account="account" :showTransferLabel="true" :show-pagination-extras="true" />
        </q-tab-panel>
        <q-tab-panel v-if="!accountPageSettings.hideContractsTab && abi" name="contract">
            <ContractTabs/>
        </q-tab-panel>
        <q-tab-panel v-if="!accountPageSettings.hideTokensTab" name="tokens">
            <TokensPanel :account="account"/>
        </q-tab-panel>
        <q-tab-panel v-if="!accountPageSettings.hideKeysTab" name="keys">
            <KeysPanel :account="account"/>
        </q-tab-panel>
        <q-tab-panel name="votes">
            <BpVotes :account="account"/>
        </q-tab-panel>
        <q-tab-panel v-if="!accountPageSettings.hideChildrenTab" name="children">
            <ChildrenPanel :account="account"/>
        </q-tab-panel>
    </q-tab-panels>
</div>
</template>

<style lang="sass">
.account-card
  width: 550px
  border-radius: 16px
  margin-top: 1.5rem
  margin-left: auto
  margin-right: auto
  margin-bottom: 2rem
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15), 0 8px 32px rgba(0, 0, 0, 0.1)
  border: 1px solid rgba(255, 255, 255, 0.08)

.tabs
  color: white
  max-width: 700px
  margin: 0 auto
  border-radius: 12px 12px 0 0
  background: rgba(255, 255, 255, 0.03)
  backdrop-filter: blur(10px)
  padding: 0 0.5rem

.header-support
  background: linear-gradient(180deg, #0A0E27 0%, #0D1229 100%) !important
  padding-bottom: 0 !important
  min-height: auto !important
  height: auto !important

// Tab panels with subtle transition
.q-tab-panels
  .q-tab-panel
    animation: fadeIn 0.3s ease

@keyframes fadeIn
  from
    opacity: 0
    transform: translateY(8px)
  to
    opacity: 1
    transform: translateY(0)
</style>

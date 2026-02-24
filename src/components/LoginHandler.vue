<script lang="ts">
import { kit, ui } from 'src/boot/wharf';
import LoginHandlerDropdown from 'src/components/LoginHandlerDropdown.vue';
import ConfigManager from 'src/config/ConfigManager';
import { useAccountStore } from 'src/stores/account';
import { computed, defineComponent, onMounted, ref } from 'vue';

export default defineComponent({
    name: 'LoginHandler',
    components: { LoginHandlerDropdown },
    setup() {
        const accountStore = useAccountStore();

        const showDropdown = ref(false);
        const account = computed(() => accountStore.accountName);
        const login = async () => {
            try {
                const result = await kit.login();
                if (result.session) {
                    accountStore.login(result.session);
                }
            } catch (e) {
                console.error('error logging in', e);
            }
        };

        onMounted(async () => {
            // Dialog element will be appended lazily when login() is called
            // appendDialogElement() causes "closer.onclick" errors on Quasar 2.18
            // Attempt to restore any existing sessions
            try {
                // This is only needed because the application state doesn't allow dynamic switching of chains
                const sessions = await kit.getSessions();
                const manager = ConfigManager.get();
                // It'll restore the first matching account for the chain once switching chains
                const matchingSession = sessions.find(session =>
                    session.chain === manager.getCurrentChain().getChainId(),
                );
                if (matchingSession) {
                    const session = await kit.restore(matchingSession);
                    if (session) {
                        accountStore.login(session);
                    }
                }
            } catch (e) {
                console.error('error restoring session', e);
            }
        });

        return {
            showDropdown,
            account,
            login,
        };
    },
});
</script>

<template>
<div>
    <LoginHandlerDropdown v-if="account"/>
    <q-btn
        v-else
        class="button-primary btn-login"
        label="Connect"
        @click="login()"
    />
</div>
</template>

<style scoped lang="sass">
.btn-login
  width: 60%
  min-width: 120px
  max-width: 140px
  height: 40px
</style>

<script lang="ts">
import { defineComponent, computed, ref } from 'vue';
import { Producer } from 'src/types';
import { getChain } from 'src/config/ConfigManager';
import { useAccountStore } from 'src/stores/account';
import { useChainStore } from 'src/stores/chain';

const chain = getChain();

const MAX_VOTE_PRODUCERS = 30;

export default defineComponent({
    name: 'ValidatorDataTable',
    props: {
        top21pay24h: { type: Number, required: true },
    },
    setup(props) {
        const accountStore = useAccountStore();
        const chainStore = useChainStore();
        const symbol = chain.getSystemToken().symbol;
        const account = computed(() => accountStore.accountName);
        const previousVote = computed(() =>
            accountStore.data.voter_info
                ? accountStore.data.voter_info.producers.map(vote =>
                    vote.toString(),
                )
                : [],
        );
        const producers = computed(() =>
            [...chainStore.producers].map(val => val.owner),
        );
        const currentVote = computed(() => {
            let votes = [...accountStore.vote];
            votes.forEach((vote, index) => {
                if (!producers.value.includes(vote)) {
                    votes.splice(index, 1);
                }
            });
            return votes;
        });
        const selection = ref<string[]>([]);
        const HeadProducer = computed(
            (): string => chainStore.head_block_producer,
        );
        const producerRows = computed(
            (): Producer[] => chainStore.producers || [],
        );
        const producerPay = computed(() => props.top21pay24h);
        const bpTop21 = computed(() => chainStore.producerSchedule);

        const maxSelected = computed(
            () => currentVote.value.length === MAX_VOTE_PRODUCERS,
        );

        const pagination = ref({
            rowsPerPage: 21,
        });
        function removeVote(index: string) {
            currentVote.value.splice(Number(index), 1);
        }
        function getLink(domain: string, username: string) {
            return `https://${domain}/${username}`;
        }

        function getFlag(alpha2: number) {
            if (alpha2) {
                return `flag-icon-${alpha2}`;
            }
            return '';
        }

        function updateVote(val: string[]) {
            val.forEach((vote, index) => {
                if (!producers.value.includes(vote)) {
                    val.splice(index, 1);
                }
            });
            accountStore.setVote(val);
        }

        function isTop21(val: string): boolean {
            return bpTop21.value.includes(val);
        }

        return {
            producerRows,
            account,
            previousVote,
            HeadProducer,
            selection,
            maxSelected,
            currentVote,
            pagination,
            producerPay,
            symbol,
            updateVote,
            removeVote,
            isTop21,
            getLink,
            getFlag,
        };
    },
});
</script>

<template>

<div class="vd-table q-pa-md">
    <div class="vd-table__list bp-list">
        <div class="vd-table__list-row q-col-gutter-sm">
            <div class="vd-table__list-col col-12">
                <q-card flat>
                    <div class="q-card-section q-pa-md text-subtitle1 text-weight-light">
                        <div class="row">
                            <div class="col-1">Rank</div>
                            <div class="col-3"> BP</div>
                            <div class="col-2 offset-1"> Rank</div>
                            <div class="col-2"> Votes</div>
                            <div class="col-2"> Reward 24h</div>
                        </div>
                    </div>
                </q-card>
            </div>
            <div
                v-for="(bp,i) in producerRows"
                :key="i"
                class="vd-table__list-col col-12"
            >
                <q-card class="producer-card" flat>
                    <div class="q-card-section">
                        <div class="row">
                            <div class="col-1 q-py-md">
                                <div class="row items-center full-height text-h6 q-px-md">{{producerRows.indexOf(bp) + 1}}</div>
                            </div>
                            <div class="col-3 q-py-md"><a class="hover-dec" :href=" '/account/' + bp.owner">
                                                           <div class="text-uppercase text-h6 text-black">{{ bp.name|| bp.owner }}</div></a>
                                <div class="text-body2">{{ bp.location }}</div>
                            </div>
                            <div class="col-2 q-py-md offset-1">
                                <div class="row items-center full-height">
                                    <q-chip
                                        v-if="HeadProducer === bp.owner "
                                        square
                                        color="primary"
                                        text-color="white"
                                        label="Producing"
                                    />
                                    <q-chip
                                        v-else-if="(producerRows.indexOf(bp) + 1) < 22"
                                        outline
                                        square
                                        color="primary"
                                        text-color="white"
                                        label="Top 21"
                                    />
                                    <q-chip
                                        v-else-if="(producerRows.indexOf(bp) + 1) < 36"
                                        outline
                                        square
                                        color="primary"
                                        text-color="white"
                                        label="Standby"
                                    />
                                    <q-chip
                                        v-else
                                        outline
                                        square
                                        color="primary"
                                        text-color="white"
                                        label="Unpaid Standby"
                                    />
                                </div>
                            </div>
                            <div class="col-2 q-py-md">
                                <div class="row items-center full-height">
                                    <div>
                                        {{ (bp.total_votes / 10000).toLocaleString(undefined, {minimumFractionDigits: 4,maximumFractionDigits: 4,}) }}
                                        <br>
                                        <q-chip
                                            v-if="bp.self_staked_boost > 0"
                                            color="primary"
                                            text-color="white"
                                            :label="`+${(bp.self_staked_boost / 10000).toLocaleString(undefined, {minimumFractionDigits: 4,maximumFractionDigits: 4,})} Boosted`"
                                            size="sm"
                                        />
                                    </div>
                                </div>
                            </div>
                            <div class="col-2 q-py-md">
                                <div class="row items-center full-height">{{ ((producerRows.indexOf(bp) + 1) < 22 ? (producerPay*((120-2*producerRows.indexOf(bp))/100)) : (producerRows.indexOf(bp) + 1) < 36 ? ((producerPay/2)*((162-2*producerRows.indexOf(bp))/100)) : 0 ).toFixed(0)  + ` ${symbol}` }}</div>
                            </div>
                            <div class="col-1 select-box q-py-md">
                                <div class="row full-selection justify-center">
                                    <q-checkbox
                                        v-model="currentVote"
                                        :val="bp.owner"
                                        :disable="!currentVote.includes(bp.owner) && currentVote.length >= 30"
                                        @update:model-value="(val)=> updateVote(val)"
                                    />
                                </div>
                                <div class="row full-selection justify-center">
                                    <q-badge v-if="previousVote.includes(bp.owner)" color="green" label="VOTED"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </q-card>
            </div>
        </div>
    </div>
</div>
</template>

<style lang="sass" scoped>
.vd-table
  &__list
    width: 100%
    &-row
      overflow-x: auto
      display: flex
      flex-direction: column
      gap: 10px
    &-col
      min-width: 1000px

.producer-card
  background: linear-gradient(180deg, #FFFFFF 0%, #FAFBFC 100%)
  border: 1px solid rgba(0, 0, 0, 0.04) !important
  border-radius: 16px !important
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1)
  overflow: hidden
  position: relative

  // Gradient top accent
  &::before
    content: ''
    position: absolute
    top: 0
    left: 0
    right: 0
    height: 3px
    background: linear-gradient(90deg, #00D4FF 0%, #4FACFE 33%, #6366F1 66%, #A855F7 100%)
    opacity: 0
    transition: opacity 0.3s ease

  &:hover
    transform: translateY(-4px)
    box-shadow: 0 12px 32px rgba(79, 172, 254, 0.15), 0 4px 12px rgba(0, 0, 0, 0.06) !important
    border-color: rgba(79, 172, 254, 0.2) !important

    &::before
      opacity: 0.8

.select-box
  background: linear-gradient(135deg, rgba(99, 102, 241, 0.04) 0%, rgba(79, 172, 254, 0.04) 100%)
  border-radius: 0 16px 16px 0
  border-left: 1px solid rgba(99, 102, 241, 0.08)
  transition: all 0.2s ease

  &:hover
    background: linear-gradient(135deg, rgba(99, 102, 241, 0.08) 0%, rgba(79, 172, 254, 0.08) 100%)

.hover-dec
  text-decoration: none
  color: #1A1A2E
  transition: all 0.2s ease
  position: relative

  &::after
    content: ''
    position: absolute
    bottom: -2px
    left: 0
    width: 0
    height: 2px
    background: linear-gradient(90deg, #4FACFE 0%, #6366F1 100%)
    transition: width 0.3s ease

  &:hover
    color: #4FACFE

    &::after
      width: 100%

// Rank number styling — gradient badge
.col-1 .text-h6
  display: inline-flex
  align-items: center
  justify-content: center
  min-width: 42px
  height: 42px
  background: linear-gradient(135deg, rgba(79, 172, 254, 0.12) 0%, rgba(99, 102, 241, 0.12) 100%)
  border-radius: 12px
  font-weight: 700
  font-size: 1rem
  letter-spacing: -0.02em
  color: #4FACFE

// BP name styling
.col-3 .text-h6
  font-weight: 700
  letter-spacing: -0.02em
  font-size: 1rem

// Location text
.text-body2
  color: rgba(0, 0, 0, 0.45)
  font-size: 0.8125rem
  font-weight: 500
  margin-top: 0.25rem

// Header row
.vd-table__list-col:first-child .q-card
  background: transparent !important
  border: none !important
  box-shadow: none !important

  .text-subtitle1
    font-weight: 600
    text-transform: uppercase
    font-size: 0.6875rem
    letter-spacing: 0.08em
    color: rgba(0, 0, 0, 0.4)

// Chip styling
:deep(.q-chip)
  border-radius: 8px !important
  font-weight: 600
  font-size: 0.6875rem
  letter-spacing: 0.04em
  transition: all 0.2s ease

  &[label="Producing"]
    background: linear-gradient(135deg, #22c55e 0%, #16a34a 100%) !important
    box-shadow: 0 2px 8px rgba(34, 197, 94, 0.4)
    animation: pulse-chip 2s ease-in-out infinite

  &[label="Top 21"]
    background: linear-gradient(135deg, #4FACFE 0%, #6366F1 100%) !important
    box-shadow: 0 2px 8px rgba(79, 172, 254, 0.3)

  &[label="Standby"]
    background: linear-gradient(135deg, rgba(79, 172, 254, 0.15) 0%, rgba(99, 102, 241, 0.15) 100%) !important
    color: #4FACFE !important
    border: 1px solid rgba(79, 172, 254, 0.3) !important

@keyframes pulse-chip
  0%, 100%
    box-shadow: 0 2px 8px rgba(34, 197, 94, 0.4)
  50%
    box-shadow: 0 2px 16px rgba(34, 197, 94, 0.6), 0 0 20px rgba(34, 197, 94, 0.3)

// Votes badge
:deep(.q-badge)
  background: linear-gradient(135deg, #22c55e 0%, #16a34a 100%) !important
  border-radius: 6px
  font-weight: 600
  font-size: 0.625rem
  letter-spacing: 0.04em
  padding: 0.25rem 0.5rem
  box-shadow: 0 2px 6px rgba(34, 197, 94, 0.3)

// Tablet responsive
@media screen and (max-width: 1100px) and (min-width: 600px)
  .vd-table__list-col
    min-width: 900px

  .producer-card
    border-radius: 14px !important

  .col-1 .text-h6
    min-width: 36px
    height: 36px
    font-size: 0.875rem

// Mobile: card layout instead of horizontal scroll
@media screen and (max-width: 599px)
  .vd-table__list-col
    min-width: unset !important
    width: 100%

  // Hide the header row on mobile
  .vd-table__list-col:first-child
    display: none

  .producer-card .q-card-section > .row
    flex-wrap: wrap !important

    // Rank
    > .col-1:first-child
      width: auto !important
      max-width: none !important
      flex: 0 0 auto !important
      padding: 0.5rem !important

      .text-h6
        min-width: 32px
        height: 32px
        font-size: 0.8rem

    // BP name
    > .col-3
      width: auto !important
      max-width: none !important
      flex: 1 1 0 !important
      padding: 0.5rem !important

      .text-h6
        font-size: 0.875rem !important

      .text-body2
        font-size: 0.75rem

    // Status chip
    > .col-2.offset-1
      margin-left: 0 !important
      width: auto !important
      max-width: none !important
      flex: 0 0 auto !important
      padding: 0.25rem 0.5rem !important

    // Votes
    > .col-2:nth-child(4)
      flex: 0 0 100% !important
      max-width: 100% !important
      padding: 0 0.5rem 0.25rem !important
      font-size: 0.8125rem

      .q-chip
        font-size: 0.6rem !important
        padding: 0.1rem 0.3rem !important

    // Reward
    > .col-2:nth-child(5)
      flex: 0 0 auto !important
      width: auto !important
      max-width: none !important
      padding: 0 0.5rem 0.5rem !important
      font-size: 0.8125rem

    // Checkbox
    > .col-1.select-box
      flex: 0 0 auto !important
      width: auto !important
      max-width: none !important
      border-radius: 0 0 16px 0 !important
      padding: 0.25rem !important
      position: absolute
      right: 0
      top: 0

  .producer-card
    position: relative

</style>

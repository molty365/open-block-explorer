<script lang="ts">
import { defineComponent, onMounted, ref } from 'vue';
import { Chart } from 'highcharts-vue';
import Highcharts from 'highcharts';
import exportingInit from 'highcharts/modules/exporting';
import { DateTuple } from 'src/types';
import { getChain } from 'src/config/ConfigManager';
import { PriceChartData } from 'src/types/PriceChartData';
import { getCssVar } from 'quasar';

const chain = getChain();

const ONE_MILLION = 1000000;
const ONE_BILLION = 1000000000;

exportingInit(Highcharts);
export default defineComponent({
    name: 'PriceChart',
    components: {
        Highcharts: Chart,
    },
    setup() {
        const hcInstance = Highcharts;
        const chartOptions = ref({
            //   uncomment to fill area
            chart: {
                type: 'area',
            },
            title: {
                text: 'Past 24h',
            },
            xAxis: {
                dateTimeLabelFormats: {
                    day: '%A, %b %e, %l %p',
                    millisecond: '%A, %b %e, %l %p',
                },
                type: 'datetime',
            },
            yAxis: {
                title: {
                    text: 'Price',
                },
            },
            legend: {
                enabled: true,
            },
            plotOptions: {
                area: {
                    // uncomment to display gradient
                    fillColor: {
                        linearGradient: {
                            x1: 0,
                            y1: 0,
                            x2: 0,
                            y2: 1,
                        },
                        stops: [
                            // [0, '#571AFF'],
                            [0, getCssVar('color-graph-shadow')],
                        ],
                    },
                    marker: {
                        radius: 0.5,
                    },
                    lineWidth: 2,
                    states: {
                        hover: {
                            lineWidth: 4,
                        },
                    },
                    threshold: null,
                },
            },
            series: [
                {
                    name: chain.getSystemToken().symbol,
                    color: getCssVar('primary'),
                    data: [] as DateTuple[],
                },
            ],
            tooltip: {
                dateTimeLabelFormats: {
                    hour: '%A, %b %e, %l %p',
                },
            },
        });
        const lastUpdated = ref(0);
        const tokenPrice = ref('');
        const marketCap = ref('');
        const rank = ref('');
        const dayVolume = ref('');
        const dayChange = ref('');

        const fetchPriceChartData = async () => {
            const data: PriceChartData = await chain.getPriceData();
            lastUpdated.value = data.lastUpdated;
            tokenPrice.value = formatCurrencyValue(data.tokenPrice);
            dayChange.value = formatPercentage(data.dayChange);
            dayVolume.value = formatCurrencyValue(data.dayVolume);
            marketCap.value = formatCurrencyValue(data.marketCap);
            chartOptions.value.series[0].data = data.prices;
        };
        const formatPercentage = (val: number): string => `${val.toFixed(2)} %`;
        const formatCurrencyValue = (val: number): string => val < 1 ? `$${val.toFixed(3)}` : val < ONE_MILLION
            ? `$${val.toFixed(2)}`
            : val < ONE_BILLION
                ? `$${(val / ONE_MILLION).toFixed(2)}M`
                : `$${(val / ONE_BILLION).toFixed(2)}B`;

        onMounted(async () => {
            await fetchPriceChartData();
        });
        return {
            hcInstance,
            chartOptions,
            lastUpdated,
            tokenPrice,
            marketCap,
            rank,
            dayVolume,
            dayChange,
            fetchPriceChartData,
            formatPercentage,
            formatCurrencyValue,
        };
    },
});
</script>

<template>
<div class="price-chart row col-12 justify-center actor-font" align="center">
    <div class="row col-11 price-box flex">
        <div class="col-xs-12 col-sx-12 col-md-8 col-lg-8 col-xs-8 q-pa-md">
            <Highcharts
                class="highcharts-description col-12"
                :options="chartOptions"
                :highcharts="hcInstance"
            />
        </div>
        <div class="col-xs-12 col-sx-12 col-md-4 col-lg-4 col-xs-4 q-pa-md">
            <div class="col-12 flex row q-mt-md">
                <div class="col-6 chart-info">
                    <p>TOKEN PRICE</p>
                    <p class="sub-title">{{ tokenPrice}}</p>
                    <p class="border-line"></p>
                </div>
                <div class="col-6 chart-info">
                    <p>MARKETCAP</p>
                    <p class="sub-title">{{ marketCap }}</p>
                    <p class="border-line"></p>
                </div>
            </div>
            <div class="col-12 flex row">
                <div class="col-6 chart-info">
                    <p>24H CHANGE</p>
                    <p class="sub-title">{{ dayChange  }}</p>
                </div>
                <div class="col-6 chart-info">
                    <p>24H VOLUME</p>
                    <p class="sub-title">{{ dayVolume }}</p>
                </div>
            </div>
        </div>
    </div>
</div>
</template>

<style lang="sass" scoped>
$medium: 750px

.chart-info
    @media screen and (max-width: $medium)
        text-align: center !important
    @media screen and (min-width: $medium)
        text-align: left !important

    p
        margin-bottom: 0.25rem
        font-size: 0.75rem
        font-weight: 600
        text-transform: uppercase
        letter-spacing: 0.05em
        color: rgba(0, 0, 0, 0.45)

.border-line
    width: 40px
    height: 3px
    background: linear-gradient(90deg, #4FACFE 0%, #6366F1 100%)
    border-radius: 2px
    margin-top: 0.5rem
    @media screen and (max-width: $medium)
        width: 100px !important
        margin: 0.5rem auto 1.5rem auto
    @media screen and (min-width: $medium)
        width: 40px

.price-box
    z-index: 1
    width: 100%
    background: #FFFFFF
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04), 0 8px 24px rgba(0, 0, 0, 0.06)
    border-radius: 16px
    border: 1px solid rgba(0, 0, 0, 0.04)
    overflow: hidden

.title
    font-style: normal
    font-weight: 600
    font-size: 14px
    line-height: 17px
    text-transform: uppercase
    color: rgba(0, 0, 0, 0.5)
    letter-spacing: 0.05em

.sub-title
    font-style: normal
    font-weight: 700
    font-size: 1.5rem
    line-height: 1.3
    color: #1A1A2E
    letter-spacing: -0.01em

.highcharts-figure,
.highcharts-data-table table
  min-width: 360px
  max-width: 800px
  margin: 1em auto

.highcharts-data-table table
  border-collapse: collapse
  border: 1px solid rgba(0, 0, 0, 0.06)
  margin: 10px auto
  text-align: center
  width: 100%
  max-width: 500px
  border-radius: 8px
  overflow: hidden

.highcharts-data-table caption
  padding: 1em 0
  font-size: 1.2em
  color: rgba(0, 0, 0, 0.6)

.highcharts-data-table th
  font-weight: 600
  padding: 0.75em

.highcharts-data-table td,
.highcharts-data-table th,
.highcharts-data-table caption
  padding: 0.75em

.highcharts-data-table thead tr,
.highcharts-data-table tr:nth-child(even)
  background: rgba(99, 102, 241, 0.03)

.highcharts-data-table tr:hover
  background: rgba(79, 172, 254, 0.08)

.highcharts-description
  height: 250px
  border-radius: 12px
  overflow: hidden
</style>

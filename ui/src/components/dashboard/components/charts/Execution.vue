<template>
    <div class="p-4">
        <div class="d-flex flex justify-content-between pb-4">
            <div>
                <p class="m-0 fs-6">
                    <span class="fw-bold">{{ t("executions") }}</span>
                    <span class="fw-light small">
                        {{ t("dashboard.per_day") }}
                    </span>
                </p>
                <p class="m-0 fs-2">
                    {{ total }}
                </p>
            </div>

            <div>
                <div>right top</div>
                <div>right bototm</div>
            </div>
        </div>
        <Bar :data="parsedData" :options="options" class="tall" />
    </div>
</template>

<script setup>
    import {computed} from "vue";
    import {useI18n} from "vue-i18n";

    import moment from "moment";
    import {Bar} from "vue-chartjs";

    import Utils from "../../../../utils/utils";
    import {
        defaultConfig,
        backgroundFromState,
        getFormat,
    } from "../../../../utils/charts.js";

    const {t} = useI18n({useScope: "global"});

    const props = defineProps({
        data: {
            type: Object,
            required: true,
        },
        total: {
            type: Number,
            required: true,
        },
    });

    const parsedData = computed(() => {
        const darkTheme = Utils.getTheme() === "dark";

        let datasets = props.data.reduce(function (accumulator, value) {
            Object.keys(value.executionCounts).forEach(function (state) {
                if (accumulator[state] === undefined) {
                    accumulator[state] = {
                        label: state,
                        backgroundColor: backgroundFromState(state),
                        yAxisID: "y",
                        data: [],
                    };
                }

                accumulator[state].data.push(value.executionCounts[state]);
            });

            return accumulator;
        }, Object.create(null));

        return {
            labels: props.data.map((r) =>
                moment(r.startDate).format(getFormat(r.groupBy)),
            ),
            datasets: [
                {
                    type: "line",
                    label: t("duration"),
                    fill: "start",
                    pointRadius: 0,
                    borderWidth: 0.2,
                    borderColor: !darkTheme ? "#7081b9" : "#7989b4",
                    yAxisID: "yB",
                    data: props.data.map((value) => {
                        return value.duration.avg === 0
                            ? 0
                            : Utils.duration(value.duration.avg);
                    }),
                },
                ...Object.values(datasets),
            ],
        };
    });

    const options = computed(() =>
        defaultConfig({
            scales: {
                x: {
                    title: {
                        display: true,
                        text: t("date"),
                    },
                    grid: {
                        display: false,
                    },
                    display: true,
                    stacked: true,
                    ticks: {
                        maxTicksLimit: 8,
                        callback: function (value) {
                            return moment(
                                new Date(this.getLabelForValue(value)),
                            ).format("MM/DD");
                        },
                    },
                },
                y: {
                    title: {
                        display: true,
                        text: t("executions"),
                    },
                    grid: {
                        display: false,
                    },
                    display: true,
                    position: "left",
                    stacked: true,
                    ticks: {
                        maxTicksLimit: 8,
                    },
                },
                yB: {
                    display: false,
                    position: "right",
                },
            },
        }),
    );
</script>

<style lang="scss" scoped>
@import "@kestra-io/ui-libs/src/scss/variables";

$height: 200px;

.tall {
    height: $height;
    max-height: $height;
}

.small {
    font-size: $font-size-xs;
    color: $gray-700;

    html.dark & {
        color: $gray-300;
    }
}
</style>

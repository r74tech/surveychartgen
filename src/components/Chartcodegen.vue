<script lang="ts">
import {
    Chart,
    ChartData,
    ChartDataset,
    ChartOptions,
    registerables,
    ChartType,
} from "chart.js";
import { defineComponent } from "vue";
import Vue from "vue";
Chart.register(...registerables);

const OPTION_PIE = {
    legend: {
        position: "bottom",
    },
    scales: {
        yAxes: [
            {
                ticks: {
                    display: false,
                    beginAtZero: true,
                },
                gridLines: {
                    display: false,
                },
            },
        ],
    },
    hoverOffset: 4,
    animation: {
        animateRotate: true,
    },
};
const OPTION_BAR = {
    indexAxis: "x",
    scales: {
        x: {
            beginAtZero: true,
        },
    },
    aspectRatio: 2,
};

const OPTION_HORIZONTAL_BAR = {
    indexAxis: "y",
    scales: {
        y: {
            beginAtZero: true,
        },
    },
    aspectRatio: 0.75,
};

function csvparse(textarea) {
    let result = {};
    // csvの解析を行う
    if (textarea.value) {
        const csvParse = (csv) =>
            csv
                .split(/\r\n|\n|\r/) //split by line feed codes
                .filter((k) => k.match(/[^,\s]/)) //remove empty lines
                .map(
                    (k) =>
                        k
                            .trim() //remove white spaces of begining and end of line
                            .replace(/,\s+/g, ",") //remove white spaces
                            .split(",") //split by cannma
                            .map((l) => (isNaN(l) ? l : parseFloat(l))) //convert string to flot
                );

        const data = csvParse(textarea.value);

        const index = data.shift(); //comment out tow lines from here
        result = {
            rawdata: data,
            index: index,
        };
    }
    return result;
}

function csv2ChartData(jsondata, select) {
    const index = jsondata.index;
    const rawdata = jsondata.rawdata;
    const labels = rawdata.map((k) => k[0]);
    const datasets = [];
    const label = index[0];
    const data = rawdata.map((k) => k[1]);
    let backgroundColor;
    switch (select.value) {
        case "pie":
            backgroundColor = [
                "#e5dbff",
                "#c5f6fa",
                "#fff3bf",
                "#ffe3e3",
                "#9775fa",
                "#69db7c",
                "#ffd43b",
                "#ff8787",
                "#6741d9",
                "#2f9e44",
                "#f08c00",
                "#f03e3e",
            ];
            break;
        default:
            backgroundColor = ["#d6d6d6"];
            break;
    }
    const ChartData: ChartData = {
        labels: labels,
        datasets: [
            {
                label: label,
                data: data,
                backgroundColor: backgroundColor,
            },
        ],
    };
    return ChartData;
}

function type2op(select, ctx, ChartData) {
    let type = "";
    let myChart;
    let mytype;
    let myoption = {};
    switch (select.value) {
        case "pie":
            myoption = OPTION_PIE;
            mytype = "pie";
            myChart = new Chart(ctx, {
                type: "pie",
                data: ChartData,
                options: myoption,
            });
            break;
        case "bar":
            myoption = OPTION_BAR;
            mytype = "bar";
            myChart = new Chart(ctx, {
                type: "bar",
                data: ChartData,
                options: myoption,
            });
            break;
        case "horizontalBar":
            myoption = OPTION_HORIZONTAL_BAR;
            mytype = "bar";
            myChart = new Chart(ctx, {
                type: "bar",
                data: ChartData,
                options: myoption,
            });
            break;
        default:
            myoption = OPTION_PIE;
            mytype = "pie";
            myChart = new Chart(ctx, {
                type: "pie",
                data: ChartData,
                options: myoption,
            });
            break;
    }
    return { chart: myChart, myoption: myoption, mytype: mytype };
}

const acc__template =
`
            <div class="accordion-item">
                <h2 class="accordion-header" id="acc{{cout__m}}--{{cout__s}}">
                    <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#col{{cout__m}}--{{cout__s}}" aria-expanded="false" aria-controls="col{{cout__m}}--{{cout__s}}">
                        <strong>{{cout__s}}. {{title}}</strong>
                    </button>
                </h2>
                <div id="col{{cout__m}}--{{cout__s}}" class="accordion-collapse collapse" aria-labelledby="acc{{cout__m}}--{{cout__s}}" data-bs-parent="#acc{{cout__m}}">
                    <div class="accordion-body">
                        <div class="chart-wrapper">
                            <canvas id="cha{{cout__m}}--{{cout__s}}"></canvas>
                        </div>
                    </div>
                </div>
            </div>
` +
`            <script>\n` +
`            {{script}}` +
`\n            <\/script>`;

const tab__template =
    `
            <div class="accordion-item">
                <h2 class="accordion-header" id="acc{{cout__m}}--{{cout__s}}">
                    <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#col{{cout__m}}--{{cout__s}}" aria-expanded="false" aria-controls="col{{cout__m}}--{{cout__s}}">
                        <strong>{{cout__s}}. {{title}}</strong>
                    </button>
                </h2>
                <div id="col{{cout__m}}--{{cout__s}}" class="accordion-collapse collapse" aria-labelledby="acc{{cout__m}}--{{cout__s}}" data-bs-parent="#acc{{cout__m}}">
                    <div class="accordion-body">
                        <ul class="nav nav-tabs" id="tabc1-1" role="tablist">
                            <li class="nav-item" role="presentation">
                                <a class="nav-link active" data-bs-toggle="tab" href="#tab{{cout__m}}--{{cout__s}}-1" id="tab{{cout__m}}--{{cout__s}}-1-tab" role="tab">全体</a>
                            </li>
                            <li class="nav-item" role="presentation">
                                <a class="nav-link" data-bs-toggle="tab" href="#tab{{cout__m}}--{{cout__s}}-2" id="tab{{cout__m}}--{{cout__s}}-2-tab" role="tab">メンバー</a>
                            </li>
                            <li class="nav-item">
                                <a class="nav-link" data-bs-toggle="tab" href="#tab{{cout__m}}--{{cout__s}}-3" id="tab{{cout__m}}--{{cout__s}}-3-tab" role="tab">非メンバー</a>
                            </li>
                        </ul>
                        <div class="tab-content">
                            <div aria-labelledby="tab{{cout__m}}--{{cout__s}}-1-tab" class="tab-pane fade show active" id="tab{{cout__m}}--{{cout__s}}-1" role="tabpanel">
                                <div class="chart-wrapper">
                                    <canvas id="cha{{cout__m}}--{{cout__s}}-1"></canvas>
                                </div>
                            </div>
                            <div aria-labelledby="tab{{cout__m}}--{{cout__s}}-2-tab" class="tab-pane fade" id="tab{{cout__m}}--{{cout__s}}-2" role="tabpanel">
                                <div class="chart-wrapper">
                                    <canvas id="cha{{cout__m}}--{{cout__s}}-2"></canvas>
                                </div>
                            </div>
                            <div aria-labelledby="tab{{cout__m}}--{{cout__s}}-3-tab" class="tab-pane fade" id="tab{{cout__m}}--{{cout__s}}-3" role="tabpanel">
                                <div class="chart-wrapper">
                                    <canvas id="cha{{cout__m}}--{{cout__s}}-3"></canvas>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
` +
`            <script>\n` +
`            {{script}}` +
`\n            <\/script>`+
`<script>\n` +
`            <\/script>`+
`<script>\n` +
`            <\/script>`;
// buttonが押されたときに関数を実行する
export default {
    setup() {
        const generate = () => {
            // textarea に入力されたjsonを取得する
            const csv = document.getElementById("csv") as HTMLTextAreaElement;
            if (!csv.value) {
                return;
            }
            const optiontype = document.getElementById(
                "ChartType"
            ) as HTMLSelectElement;
            const ctx = document.getElementById(
                "chart-container"
            ) as HTMLCanvasElement;
            let chartStatus = Chart.getChart("Chart"); // <canvas> id
            // console.log(chartStatus);
            if (chartStatus != undefined) {
                chartStatus.destroy();
            }
            const domchart = document.getElementById("Chart") as HTMLCanvasElement;
            const color = document.getElementById("color") as HTMLInputElement;

            const data = csvparse(csv);
            const ChartData = csv2ChartData(data, optiontype);
            if (color.value) {
                ChartData.datasets[0].backgroundColor = color.value
                    .replace(/\s/g, "")
                    .replace(/\"/g, "")
                    .split(",");
            }
            let myChart = type2op(optiontype, domchart, ChartData);
            let myoption = myChart["myoption"];
            let mytype = myChart["mytype"];
            let title = ChartData["datasets"][0]["label"];
            ChartData["datasets"][0]["label"] = "回答数";
            let textChartData = JSON.stringify(ChartData);
            let textOption = JSON.stringify(myoption);
            let cout__m = document.getElementById("cout__m") as HTMLInputElement;
            let cout__s = document.getElementById("cout__s") as HTMLInputElement;
            let codecontent = `new Chart($("#cha"), {
                type: "${mytype}",
                data: ${textChartData},
                options:${textOption},
            });`;
            let code = document.getElementById("code") as HTMLTextAreaElement;
            let usetemplate = document.getElementById(
                "usetemplate"
            ) as HTMLInputElement;
            let templatetype, t_type;
            switch (usetemplate.value) {
                case "acc":
                    templatetype = acc__template;
                    t_type = 1;
                    break;
                case "tab":
                    templatetype = tab__template;
                    t_type = 2;
                    break;
                default:
                    templatetype = acc__template;
                    t_type = -1;
                    break;
            }
            let template = templatetype
                .replace(/{{cout__m}}/g, cout__m.value)
                .replace(/{{cout__s}}/g, cout__s.value)
                .replace(/{{title}}/g, title)
                .replace(
                    /{{script}}/g,
                    codecontent.replace(
                        "#cha",
                        "#cha" + cout__m.value + "--" + cout__s.value
                    )
                );
            // console.log(template);
            switch (t_type) {
                case 1:
                    code.value = template;
                    break;
                case 2:
                    code.value = template;
                    break;
                case -1:
                    code.value = codecontent;
                    break;
            }

            // cout__sの値を1増やす
            cout__s.value = (parseInt(cout__s.value) + 1).toString();
        };

        return {
            generate,
        };
    },
};
</script>
  
<template>
    <div class="wrapper">
        <div class="field section">
            <p class="control columns">
                <textarea id="csv" class="textarea" placeholder="csv"></textarea>
            </p>
            <p class="control columns">
                <span class="select">
                    <select id="ChartType">
                        <option value="pie">PieChart</option>
                        <option value="bar">BarChart</option>
                        <option value="horizontalBar">HorizontalBarChart</option>
                    </select>
                </span>
                <button @click="generate" class="button is-info is-outlined">
                    表示
                </button>
            </p>
            <p class="control columns">
                <span class="select">
                    <select id="usetemplate">
                        <option>template None</option>
                        <option selected value="acc">accordion-item</option>
                        <option value="tab">tab</option>
                    </select>
                </span>
            </p>
            <p class="control columns">
                <label for="cont__m">大項目:</label>
                <input id="cout__m" class="input column is-two-fifths" type="text" placeholder="main count" value="1" name="cout__m" />
                <label for="cout__s">中項目:</label>
                <input id="cout__s" class="input column is-two-fifths" type="text" placeholder="sub count" value="1" name="cout__s" />
            </p>
            <p class="control columns">
                <label for="color">色:</label>
                <input id="color" class="input column is-four-fifths" type="text" placeholder='"#96f2d7", "#0ca678",' name="color" />
            </p>
        </div>
        <div class="highlight">
            <pre><textarea id="code" class="textarea"></textarea></pre>
        </div>

        <div id="chart-container">
            <canvas id="Chart"></canvas>
        </div>
    </div>
</template>

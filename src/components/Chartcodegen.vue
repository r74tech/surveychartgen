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
  let myoption = {};
  switch (select.value) {
    case "pie":
      myoption = OPTION_PIE;
      myChart = new Chart(ctx, {
        type: "pie",
        data: ChartData,
        options: myoption,
      });
      break;
    case "bar":
      myoption = OPTION_BAR;
      myChart = new Chart(ctx, {
        type: "bar",
        data: ChartData,
        options: myoption,
      });
      break;
    case "horizontalBar":
      myoption = OPTION_HORIZONTAL_BAR;
      myChart = new Chart(ctx, {
        type: "bar",
        data: ChartData,
        options: myoption,
      });
      break;
    default:
      myoption = OPTION_PIE;
      myChart = new Chart(ctx, {
        type: "pie",
        data: ChartData,
        options: myoption,
      });
      break;
  }
  return { chart: myChart, myoption: myoption };
}

// buttonが押されたときに関数を実行する
export default {
  setup() {
    const generate = () => {
      // textarea に入力されたjsonを取得する
      const csv = document.getElementById("csv") as HTMLTextAreaElement;
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
      const data = csvparse(csv);
      const ChartData = csv2ChartData(data, optiontype);
      let myChart = type2op(optiontype, domchart, ChartData);
      let myoption = myChart["myoption"];
      console.log(myoption);
      let textChartData = JSON.stringify(ChartData);
      let textOption = JSON.stringify(myoption);
      let codecontent = `new Chart($("#cha"), {
                    type: "${optiontype.value}",
                    data: ${textChartData},
                    options:${textOption},
                  });`;
      let code = document.getElementById("code") as HTMLTextAreaElement;
      code.innerHTML = codecontent.replace(/                  /g, "");
    };

    return {
      generate,
    };
  },
};
</script>
  
<template>
  <div class="wrapper">
    <textarea id="csv" class="textarea" placeholder="csv"></textarea>
    <div class="select">
      <select id="ChartType">
        <option value="pie">PieChart</option>
        <option value="bar">BarChart</option>
        <option value="horizontalBar">HorizontalBarChart</option>
      </select>
    </div>
    <button @click="generate" class="button is-info is-outlined">
      表示
    </button>
    <div class="highlight">
      <pre><code id="code"></code></pre>
    </div>

    <div id="chart-container">
      <canvas id="Chart"></canvas>
    </div>
  </div>
</template>

<template>
  <div id="spc_chart" class="spc_chart" style="height:100%"/>
</template>

<script>
import Plotly from "plotly.js-dist";
// import twLocale from "plotly.js-locales/zh-tw";
// import cnLocale from "plotly.js-locales/zh-cn";

export default {
  name: "spc_chart",
  props: {
    chartTitle: {
      type: String,
      default: "SPC Chart"
    },
    chartData: {
      type: Object,
      default: () => ({
        x: [0], // 時間序列
        y: [0] //defect 數量
      })
    },
    ucl: {
      // 管制上限
      type: Number,
      default: 5
    },
    lcl: {
      // 管制下限
      type: Number,
      default: -5
    },
    cl: {
      // 中位數
      type: Number,
      default: 0
    }
  },
  data: () => ({
    theme: {
      primary: "#2196F3",
      secondary: "#82B1FF",
      accent: "#82B1FF",
      error: "#FF5252",
      info: "#2196F3",
      success: "#4CAF50",
      warning: "#FFC107",
      background: "#E3F2FD",
      gridColor: "#AAAAAA",
      text: "#263238"
    }
  }),
  watch: {
    chartData: {
      handler() {
        this.drawPlot();
      },
      deep: true
    },
    ucl() {
      this.drawPlot();
    },
    lcl() {
      this.drawPlot();
    },
    cl() {
      this.drawPlot();
    }
  },
  methods: {
    myEventHandler() {
      this.drawPlot();
    },
    drawPlot() {
      // let offset = 0.75;
      // if (this.$i18n.locale === "tw") {
      //   Plotly.register(twLocale);
      //   Plotly.setPlotConfig({locale: 'zh-TW'})
      // } else if (this.$i18n.locale === "cn") {
      //   Plotly.register(cnLocale);
      //   Plotly.setPlotConfig({locale: 'zh-CN'})
      // }

      const selectorOptions = {
        buttons: [
          {
            step: "month",
            // stepmode: 'backward',
            count: 1,
            label: "1m"
          },
          {
            step: "month",
            stepmode: "backward",
            count: 6,
            label: "6m"
          },
          {
            step: "year",
            stepmode: "todate",
            count: 1,
            label: "YTD"
          },
          {
            step: "year",
            stepmode: "backward",
            count: 1,
            label: "1y"
          },
          {
            step: "all"
          }
        ],
        yanchor: "bottom",
        bgcolor: this.theme.secondary,
        bordercolor: this.theme.text,
        font: {
          family: "Courier",
          size: 16,
          color: this.theme.text
        }
      };

      const Data = {
        type: "scatter",
        x: this.chartData.x,
        y: this.chartData.y,
        mode: "lines+markers",
        name: "spc.DATA",
        showlegend: true,
        hoverinfo: "all",
        line: {
          // color: "blue",
          width: 2
        },
        marker: {
          // color: "blue",
          size: 8,
          symbol: "circle"
        }
      };

      const Viol = {
        type: "scatter",
        x: this.dataViol.x,
        y: this.dataViol.y,
        mode: "markers",
        name: "VIOLATION",
        showlegend: true,
        marker: {
          color: "rgb(255,65,54)",
          line: { width: 5 },
          opacity: 1,
          size: 14,
          symbol: "circle-open"
        }
      };

      const CL = {
        type: "scatter",
        x: [
          this.chartData.x[0],
          this.dataLength,
          null,
          this.chartData.x[0],
          this.dataLength
        ],
        dx: 0,
        y: [this.lcl, this.lcl, null, this.ucl, this.ucl],
        mode: "lines",
        name: "spc.LCL_UCL",
        showlegend: true,
        line: {
          color: "red",
          width: 2,
          dash: "dash"
        }
      };

      const Center = {
        type: "scatter",
        x: [this.chartData.x[0] || 0, this.dataLength],
        y: [0, 0],
        mode: "lines",
        name: "spc.CL",
        showlegend: true,
        line: {
          color: this.theme.info,
          width: 2
        }
      };

      const CCL = [
        {
          type: "line",
          xref: "paper",
          x0: 0,
          x1: 1,
          y0: this.cl,
          y1: this.cl,
          line: {
            color: this.theme.info,
            width: 2
          }
        },
        {
          type: "line",
          xref: "paper",
          x0: 0,
          y0: this.lcl,
          x1: 1,
          y1: this.lcl,
          line: {
            color: "red",
            width: 2,
            dash: "dash"
          }
        },
        {
          type: "line",
          xref: "paper",
          x0: 0,
          y0: this.ucl,
          x1: 1,
          y1: this.ucl,
          line: {
            color: "red",
            width: 2,
            dash: "dash"
          }
        }
      ];

      const data = [Data, Viol, CL, Center];

      const layout = {
        title: this.chartTitle,
        shapes: CCL,
        xaxis: {
          zeroline: false,
          gridcolor: this.theme.gridColor,
          rangeselector: selectorOptions,
          tickformat: "%Y-%m-%d\n%H:%M:%S",
        },
        yaxis: {
          zeroline: false,
          gridcolor: this.theme.gridColor,
          range: [this.dataMinMax.min, this.dataMinMax.max]
        },
        template: {
          layout: {
            font: {
              family: "Courier",
              size: 18,
              color: this.theme.text
            },
            paper_bgcolor: this.theme.background,
            plot_bgcolor: this.theme.background
          }
        }
      };

      const options = {
        displaylogo: false
      };

      Plotly.newPlot("spc_chart", data, layout, options);
    }
  },
  computed: {
    dataLength() {
      return Math.abs(Math.ceil(Math.max(...this.chartData.x) * 1.2));
    },
    dataMinMax() {
      const min = Math.min(...this.chartData.y, this.lcl);
      const max = Math.max(...this.chartData.y, this.ucl);
      return {
        min: min - Math.abs(Math.ceil(min*1.2)),
        max: max + Math.abs(Math.ceil(max*1.2))
      };
    },
    dataViol() {
      let x = [];
      let y = [];

      this.chartData.y.reduce((acc, value, index) => {
        if (value > this.ucl || value < this.lcl) {
          x.push(this.chartData.x[index]);
          y.push(value);
        }
      });

      return { x, y };
    }
  },
  mounted() {
    this.drawPlot();
  },
  created() {
    window.addEventListener("resize", this.myEventHandler);
  },
  destroyed() {
    window.removeEventListener("resize", this.myEventHandler);
  }
};
</script>

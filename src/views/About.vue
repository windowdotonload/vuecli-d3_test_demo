<!--
 * @Descripttion: 
 * @version: 
 * @Author: windowdotonload
-->
<template>
  <div class="about">
    <div :style="hoverbox">{{ abstract }}</div>
    <svg width="1220" height="600"></svg>
  </div>
</template>

<script>
import * as d3 from "d3";
export default {
  data() {
    return {
      abstract: "",
      hoverbox: {
        left: "",
        top: "",
        position: "absolute",
        width: "100px",
        height: "100px",
        border: "1px solid black",
        background: "red",
        "z-index": "999",
      },
      recnodes: [
        { name: "湖南邵阳", color: "red", flag: 1 },
        { name: "山东莱州", color: "black" },
        { name: "广东阳江" },
        { name: "山东枣庄" },
        { name: "泽" },
        { name: "恒" },
        { name: "鑫" },
        { name: "明山" },
        { name: "班长" },
      ],
      new: [
        { name: "湖南邵阳", color: "red" },
        { name: "山东莱州", color: "black" },
        { name: "广东阳江" },
        { name: "山东枣庄" },
        { name: "泽" },
        { name: "恒" },
        { name: "鑫" },
        { name: "明山" },
        { name: "班长" },
        { name: "new" },
      ],
      new1: [
        { source: 0, target: 4, relation: "籍贯", value: 1.8 },
        { source: 4, target: 5, relation: "舍友", value: 1 },
        { source: 4, target: 6, relation: "舍友", value: 1 },
        { source: 4, target: 7, relation: "舍友", value: 1 },
        { source: 1, target: 6, relation: "籍贯", value: 2 },
        { source: 2, target: 5, relation: "籍贯", value: 0.9 },
        { source: 3, target: 7, relation: "籍贯", value: 1 },
        { source: 5, target: 6, relation: "同学", value: 1.6 },
        { source: 6, target: 7, relation: "朋友", value: 0.7 },
        { source: 6, target: 8, relation: "职责", value: 2 },
        { source: 0, target: 9, relation: "new", value: 2 },
      ],
      rededges: [
        { source: 0, target: 4, relation: "籍贯", value: 1.8 },
        { source: 4, target: 5, relation: "舍友", value: 1 },
        { source: 4, target: 6, relation: "舍友", value: 1 },
        { source: 4, target: 7, relation: "舍友", value: 1 },
        { source: 1, target: 6, relation: "籍贯", value: 2 },
        { source: 2, target: 5, relation: "籍贯", value: 0.9 },
        { source: 3, target: 7, relation: "籍贯", value: 1 },
        { source: 5, target: 6, relation: "同学", value: 1.6 },
        { source: 6, target: 7, relation: "朋友", value: 0.7 },
        { source: 6, target: 8, relation: "职责", value: 2 },
      ],
      force: {},
    };
  },
  created() {},
  mounted() {
    console.log("d3", d3);
    console.log(d3.select);
    console.log(d3.select("svg").attr);
    console.log(d3.select("svg").attr("width"));

    this.init();
  },
  methods: {
    init() {
      console.log("---------ok-------------");
      var marge = { top: 60, bottom: 60, left: 20, right: 60 };
      var svg = d3.select("svg");
      console.log("svg", svg);
      var width = svg.attr("width");
      var height = svg.attr("height");
      console.log("width", width);
      console.log("height", height);
      var g = svg
        .append("g")
        .attr("transform", "translate(" + marge.top + "," + marge.left + ")");

      var marker = svg
        .append("marker")
        //.attr("id", function(d) { return d; })
        .attr("id", "resolved")
        //.attr("markerUnits","strokeWidth")//设置为strokeWidth箭头会随着线的粗细发生变化
        .attr("markerUnits", "userSpaceOnUse")
        .attr("viewBox", "0 -5 10 10") //坐标系的区域
        .attr("refX", 18) //箭头坐标 应为circle也有半径，所示refX设置为18，要看circle的半径大小
        .attr("refY", 0)
        .attr("markerWidth", 12) //标识的大小
        .attr("markerHeight", 12)
        .attr("orient", "auto") //绘制方向，可设定为：auto（自动确认方向）和 角度值
        .attr("stroke-width", 2) //箭头宽度
        .append("path")
        .attr("d", "M0,-3.5L10,0L0,3.5") //箭头的路径
        .attr("fill", "#000000")
        .attr("stysle", "opacity:0.5"); //箭头颜色
      //准备数据
      var nodes = this.recnodes;

      var edges = this.rededges;
      //设置一个color的颜色比例尺，为了让不同的扇形呈现不同的颜色
      var colorScale = d3
        .scaleOrdinal()
        .domain(d3.range(nodes.length))
        .range(d3.schemeCategory10);
      // console.log('colorScalse', colorScale)
      // console.log('d3.range', d3.range(nodes.length))
      // console.log('d3.schemeCategory10', d3.schemeCategory10)
      //新建一个力导向图
      var forceSimulation = d3
        .forceSimulation()
        .force("link", d3.forceLink())
        .force("charge", d3.forceManyBody())
        .force("center", d3.forceCenter())
        .force("collision", d3.forceCollide(25));
      // console.log('----------------')
      // console.log(forceSimulation)

      //初始化力导向图，也就是传入数据
      //生成节点数据
      forceSimulation.nodes(nodes).on("tick", ticked); //这个函数很重要，后面给出具体实现和说明
      // console.log('--------')
      // console.log('nodes', nodes)
      // console.log('--------')

      //生成边数据
      forceSimulation
        .force("link")
        .links(edges)
        .id((d) => {
          return nodes.name;
        })
        .distance(function (d) {
          //每一边的长度
          return d.value * 150;
        });
      // console.log('edges', edges)
      //设置图形的中心位置
      forceSimulation
        .force("center")
        .x(width / 2)
        .y(height / 2);
      //在浏览器的控制台输出
      console.log(nodes);
      console.log(edges);

      //有了节点和边的数据后，我们开始绘制
      //绘制边
      var links = g
        .append("g")
        .selectAll("line")
        .data(edges)
        .enter()
        .append("line")
        .attr("stroke", function (d, i) {
          // console.log(d);

          return colorScale(i);
        })
        .attr("stroke-width", 1)
        .attr("marker-end", "url(#resolved)");

      var linksText = g
        .append("g")
        .selectAll("text")
        .data(edges)
        .enter()
        .append("text")
        .text(function (d) {
          return d.relation;
        });

      //绘制节点
      //老规矩，先为节点和节点上的文字分组
      var gs = g
        .selectAll(".circleText")
        .data(nodes)
        .enter()
        .append("g")
        .attr("transform", function (d, i) {
          var cirX = d.x;
          var cirY = d.y;
          return "translate(" + cirX + "," + cirY + ")";
        })
        .call(
          d3.drag().on("start", started).on("drag", dragged).on("end", ended)
        );

      //绘制节点
      gs.append("circle")
        .attr("r", 10)
        .attr("style", function (d, i) {
          if (d.flag) {
            if (d.flag == 1) {
              return "cursor:pointer"; //让鼠标悬浮变为手指
            }
          }
        })
        .attr("fill", function (d, i) {
          console.log("d", d);
          if (d.flag) {
            if (d.flag == 1) {
              return "orange";
            }
          }
          // return colorScale(i);
          return "skyblue";
        })
        .on("click", this.nodesclick)
        .on("mouseover", this.mousehover)
        .on("mouseout", this.mouseout);

      //文字
      gs.append("text")
        .attr("x", -10)
        .attr("y", -20)
        .attr("dy", 10)
        .text(function (d) {
          return d.name;
        });

      function ticked() {
        links
          .attr("x1", function (d) {
            return d.source.x;
          })
          .attr("y1", function (d) {
            return d.source.y;
          })
          .attr("x2", function (d) {
            return d.target.x;
          })
          .attr("y2", function (d) {
            return d.target.y;
          });

        linksText
          .attr("x", function (d) {
            return (d.source.x + d.target.x) / 2;
          })
          .attr("y", function (d) {
            return (d.source.y + d.target.y) / 2;
          });

        gs.attr("transform", function (d) {
          return "translate(" + d.x + "," + d.y + ")";
        });
      }
      function started(d) {
        // console.log(d3.event);
        // debugger;
        if (!d3.event.active) {
          forceSimulation.alphaTarget(0.8).restart();
        }
        d.fx = d.x;
        d.fy = d.y;
      }
      function dragged(d) {
        d.fx = d3.event.x;
        d.fy = d3.event.y;
      }
      function ended(d) {
        if (!d3.event.active) {
          forceSimulation.alphaTarget(0);
        }
        d.fx = null;
        d.fy = null;
      }

      this.force = forceSimulation;
    },

    nodesclick(d) {
      // window.location.reload();
      // console.log(this.force);
      // console.log(d);
      // console.log("123");
      // console.log("this", this);
      // this.recnodes.push({ name: "new nodes" });
      // debugger;
      // console.log(this.nodes.length - 1);
      // this.recedges.push({
      //   source: d.index,
      //   target: this.nodes.length - 1,
      //   relation: "new",
      //   value: 2,
      // });
      console.log("d  ", d);
      if (d.flag) {
        if (d.flag == 1) {
          let svg = document.getElementsByTagName("svg")[0];
          let g = svg.getElementsByTagName("g")[0];
          svg.removeChild(g);
          this.force.stop();
          this.recnodes = this.new;
          this.rededges = this.new1;
          this.init();
        }
      }
    },
    mousehover(d) {
      console.log("mouseover");
      console.log(d);
      this.abstract = d.name;
      // this.hoverbox.visibility = "visible";
      // this.hoverbox.left = `${d.x + 100}px`;
      // this.hoverbox.top = `${d.y + 100}px`;
      this.hoverbox = {
        "border-radius": "5px",
        left: `${d.x + 100}px`,
        top: `${d.y + 100}px`,
        position: "absolute",
        opacity: 1,
        width: "100px",
        height: "100px",
        border: "1px solid black",
        background: "red",
        "z-index": "999",
      };
    },
    mouseout(d) {
      console.log("mouseover");
      // this.hoverbox.visibility = "hidden";
      this.hoverbox = {
        position: "absolute",
        opacity: 0,
        width: "100px",
        height: "100px",
        border: "1px solid black",
        background: "red",
        "z-index": "999",
      };
    },
  },
};
</script>

<style lang="scss" scoped>
</style>

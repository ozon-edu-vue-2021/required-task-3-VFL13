<template>
  <div class="map">
    <h3>Карта офиса</h3>

    <div v-if="!isLoading" class="map-root">
      <!-- map -->
      <MapSVG ref="mapSvg" />
      <WorkPlaceSVG ref="placeSVG" v-show="false" />
    </div>
    <div v-else>Loading...</div>
  </div>
</template>

<script>
import * as d3 from "d3";
import tables from "../assets/data/tables.json";
import legend from "../assets/data/legend.json";
import MapSVG from "../assets/images/map.svg";
import WorkPlaceSVG from "../assets/images/workPlace.svg";

export default {
  components: {
    MapSVG,
    WorkPlaceSVG,
  },
  data() {
    return {
      isLoading: false,
      svgMap: null,
      gMap: null,
      svgWorkPlace: null,
      workPlaces: [],
    };
  },
  mounted() {
    // Выбираем svg с картой и вешаем обработчик клика.
    this.svgMap = d3.select(this.$refs.mapSvg);
    this.svgMap.on("click", this.clickBG);
    this.gMap = this.svgMap.select("g");
    this.workPlaces = tables;

    this.svgWorkPlace = d3.select(this.$refs.placeSVG);

    if (this.gMap) {
      this.drawTable();
    } else {
      console.error("map svg don't have g element");
    }
  },
  methods: {
    clickBG() {
      this.$emit("hide-user-info");
    },
    clickWP(user_id) {
      // При клике на рабочее место, передаем его ID.
      this.$emit("show-user-info", user_id);
    },
    drawTable() {
      // создаем группу рабочих мест

      const svgWorkPlacesGroup = this.gMap
        .append("g")
        .classed("groupPlaces", true);

      // В группе размещаем рабочие места, стилизуем, добавляем обработчик клика.

      this.workPlaces.map((place) => {
        svgWorkPlacesGroup
          .append("g")
          .attr(
            "transform",
            `translate(${place.x}, ${place.y}) rotate(${
              place.rotate || 0
            }) scale(0.5)`
          )
          .attr("id", place.id)
          .classed("employer-place", true)
          .attr("group_id", place.group_id)
          .html(this.svgWorkPlace.html())
          .attr(
            "fill",
            legend.find((it) => it.group_id === place.group_id)?.color
          )
          .on("click", (event) => {
            event.stopPropagation();
            this.clickWP(place._id);
          });
      });
    },
  },
};
</script>

<style scoped>
.map {
  height: 100%;
  width: 100%;
  padding: 24px;
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

.map-root {
  height: 100%;
  width: 100%;
  overflow: hidden;
  box-sizing: border-box;
}

h3 {
  margin-top: 0;
}

::v-deep svg {
  height: 100%;
  width: 100%;
}

::v-deep .table {
  cursor: pointer;
}
</style>

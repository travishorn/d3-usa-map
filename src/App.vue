<script setup>
import { onMounted } from "vue";
import { select } from "d3-selection";
import { geoNaturalEarth1, geoAlbersUsa, geoStereographic, geoPath } from "d3-geo";
import admin1 from "./assets/admin1.json";

const countryAlpha3 = "USA";
const svgId = `map-${crypto.randomUUID()}`;
const width = 1000;
const height = 900;

const country = {
    type: "FeatureCollection",
    features: admin1.features.filter((feature) => {
      return feature.properties["ISO3166-1-Alpha-3"] === countryAlpha3;
    }),
  };

const albersUsa = geoAlbersUsa()
  .fitExtent([[0, 0], [width, height]], country);

const stereographic = geoStereographic()
  .fitExtent([[0, 0], [width, height]], country);

const naturalEarth = geoNaturalEarth1()
  .fitExtent([[0, 0], [width, height]], country);

const path = geoPath();

switch (countryAlpha3) {
  case "USA":
    path.projection(albersUsa)
    break;
  case "ATA":
    path.projection(stereographic)
    break;
  default:
    path.projection(naturalEarth);
}

onMounted(() => {
  const svg = select(`#${svgId}`);

  if(country.features.length) {
    svg.selectAll("path.boundary")
      .data(country.features)
      .join("path")
      .attr("class", "boundary")
      .attr("fill", "#EBEBEB")
      .attr("stroke-width", 1)
      .attr("stroke", "white")
      .attr("d", path);

    svg.selectAll("text.admin1Label")
      .data(country.features)
      .join("text")
      .attr("class", "admin1Label")
      .text(d => d.properties.name)
      .attr("transform", d => `translate(${path.centroid(d)})`)
      .attr("font-family", "Helvetica, Arial, sans-serif")
      .attr("font-weight", 500)
      .attr("text-anchor", "middle")
      .attr("alignment-baseline", "middle")
      .attr("opacity", 0.6)
      .attr("font-size", 12)
      .attr("letter-spacing", 3);

    const t = svg.append("text")
      .text(country.features[0].properties.country.toUpperCase())
      .attr("font-family", "Helvetica, Arial, sans-serif")
      .attr("font-weight", 500)
      .attr("text-anchor", "middle")
      .attr("alignment-baseline", "middle")
      .attr("opacity", 0.15)
      .attr("font-size", 35)
      .attr("letter-spacing", 24)
      .attr("transform-origin", `${width / 2} ${height / 2}`);

    const bb = t.node().getBBox();

    t.attr("transform", `scale(${width / bb.width}) translate(${width / 2}, ${height / 2})`);
  } else {
    svg.append("text")
      .text("UNKNOWN")
      .attr("transform", `translate(${width / 2}, ${height / 2})`)
      .attr("font-family", "Helvetica, Arial, sans-serif")
      .attr("font-weight", 500)
      .attr("text-anchor", "middle")
      .attr("alignment-baseline", "middle")
      .attr("opacity", 0.15)
      .attr("font-size", 80)
      .attr("letter-spacing", 24);
  }
});
</script>

<template>
  <div class="container">
    <svg :id="svgId" :viewBox="`0 0 ${width} ${height}`"></svg>
  </div>
</template>

<style>
.container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 40px;
}
</style>
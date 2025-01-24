<script lang="ts">
    import * as d3 from "d3";
    import data from "/src/assets/data.json";
    import {StringBuilder} from "./util/StringBuilder.ts";
    import {onMount} from "svelte";

    onMount(async () => {
        // Render graph
        const graph = d3.select("#graph")
            .style("fill", "none")
            .style("stroke", "black");

        const height = 1200;
        const width = 1200;

        const treeLayout = d3.tree()
            .size([height, width]);

        graph
            .attr("height", height + 80) // + 80 to allow the last row of images to display... horrible...
            .attr("width", width);

        const root = d3.hierarchy(data);
        const links = treeLayout(root).links();
        const linkGen = d3.linkVertical()
            .x(d => d.x)
            .y(d => d.y);

        graph.selectAll("path")
            .data(links)
            .enter().append("path")
            .attr("d", linkGen);

        // Tooltip functions
        let tooltip = d3.select("#container")
            .append("div")
            .style("position", "absolute")
            .style("z-index", "100")
            .style("opacity", 0)
            .attr("class", "tooltip")
            .style("background-color", "lightgray")
            .style("border", "solid")
            .style("border-width", "2px")
            .style("border-radius", "5px")
            .style("padding", "5px");

        let renderHtml = (datum) => {
            let sb = new StringBuilder();
            sb.write("<div class=\"w-max\">");
            sb.write("<a href=" + datum.data.dbLink + ">" + datum.data.generation + ". " + datum.data.name + " (" + datum.data.nameJp + ")" + "</a>");
            sb.write("<br>");
            sb.write("Promoted: " + datum.data.promoted);
            sb.write("<br>");
            if (datum.data.style) {
                sb.write("Style: " + datum.data.style);
                sb.write("<br>")
            }
            sb.write("Stable: " + datum.data.stable);
            sb.write("<br>");
            sb.write("Ichimon: " + datum.data.ichimon);
            sb.write("<br>");
            if (datum.data.relationship) {
                sb.write("Relationship: " + datum.data.relationship);
                sb.write("<br>");
            }
            if (datum.data.notes) {
                sb.write("Notes: <i>" + datum.data.notes + "</i>");
                sb.write("<br>");
            }
            sb.write("</div>");

            return sb.toString();
        }

        let mouseover = _ => tooltip.style("opacity", 1);

        let mousemove = (event, datum) =>
            tooltip
                .html(renderHtml(datum))
                .style("left", event.offsetX + "px")
                .style("top", event.offsetY + "px");

        let mouseleave = _ => tooltip.style("opacity", 0);

        let click = (_, datum) => window.open(datum.data.dbLink, "_blank").focus();

        // Render tree nodes
        graph.selectAll(".node")
            .data(root.descendants())
            .enter()
            .append("image")
            .attr("class", d => "node" + (d.children ? "node--internal" : "node--leaf"))
            .attr("class", "rikishi")
            .attr("transform", d => `translate(${d.x},${d.y})`)
            .attr("id", d => "rikishi" + d.data.generation)
            .attr("xlink:href", d => "/src/assets/img/rikishi/" + d.data.image + ".jpg")
            .attr("x", "-40px")
            .attr("y", "0px")
            .attr("width", "80px")
            .attr("height", "80px")
            .style("clip-path", "circle()")
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave)
            .on("click", click);
    });
</script>

<div id="container" class="p-4 grid place-items-center">
    <svg id="graph"></svg>
</div>
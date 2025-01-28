<script lang="ts">
    import * as d3 from "d3";
    import data from "./assets/data.json";
    import { StringBuilder } from "./lib/StringBuilder.ts";
    import { onMount } from "svelte";

    onMount(async () => {

        // Render graph
        const graph = d3.select("#graph")
            .style("fill", "none")
            .style("stroke", "black")
            .style("stroke-width", "3px");

        const height = 1200;
        const width = 1200;
        const imageSize = 80; // in px, size of images as rendered on top of graph nodes
        const nodeOffset = 80 / 2; // this offset is used in various places to align images/tooltip bounds on top of graph nodes

        const treeLayout = d3.tree()
            .size([height, width]);

        graph
            .attr("height", height + imageSize) // allows the last row of images to display
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

        // Tooltip
        let tooltip = d3.select("#graph-container")
            .append("div")
            .style("position", "absolute")
            .style("opacity", 0)
            .attr("class", "tooltip")
            .attr("z-index", 100)
            .style("background-color", "lightgray")
            .style("border", "solid")
            .style("border-width", "2px")
            .style("border-radius", "5px")
            .style("padding", "5px");

        // Tooltip functions
        let renderHtml = (datum) => {
            let sb = new StringBuilder();
            sb.write("<div class=\"w-full\">");
            sb.write("<a class=\"font-bold\" href=" + datum.data.dbLink + ">" + datum.data.generation + ". " + datum.data.name + " (" + datum.data.nameJp + ")" + "</a>");
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
            if (datum.data.myoseki) {
                sb.write("Myōseki: " + datum.data.myoseki);
                sb.write("<br>");
            }
            if (datum.data.relationship) {
                sb.write("Relationship: " + datum.data.relationship);
                sb.write("<br>");
            }
            if (datum.data.notes) {
                sb.write("<p class=\"w-full\">Notes: <i>" + datum.data.notes + "</i></p>");
                // sb.write("<br>");
            }
            sb.write("</div>");

            return sb.toString();
        }

        let mouseover = _ => tooltip.style("opacity", 1);

        let mousemove = (event, datum) =>
            tooltip
                .html(renderHtml(datum))
                .style("left", event.pageX + "px")
                .style("top", (event.pageY + nodeOffset) + "px");

        let mouseleave = _ => tooltip.style("opacity", 0);

        let click = (_, datum) => window.open(datum.data.dbLink, "_blank").focus();

        // Add circle border to nodes denoting ceremony style by colour
        graph.selectAll(".node")
            .data(root.descendants())
            .enter()
            .append("circle")
            .attr("transform", d => `translate(${d.x},${d.y})`)
            .attr("r", "41")
            .attr("cx", "0")
            .attr("cy", nodeOffset)
            .style("stroke", d => {
                let borderColour = "#6d6d6d" // default where style is unknown
                if (d.data.style == "Unryū") {
                    borderColour = "#af4f30";
                } else if (d.data.style == "Shiranui") {
                    borderColour = "#498e7b";
                }
                return borderColour;
            });

        // Render tree nodes
        graph.selectAll(".node")
            .data(root.descendants())
            .enter()
            .append("image")
            .attr("class", d => "node" + (d.children ? "node--internal" : "node--leaf"))
            .attr("class", "rikishi")
            .attr("transform", d => `translate(${d.x},${d.y})`)
            .attr("id", d => "rikishi" + d.data.generation)
            .attr("xlink:href", d => "./img/rikishi/" + d.data.image + ".jpg")
            .attr("x", "-" + nodeOffset + "px")
            .attr("y", "0px")
            .attr("width", "80px")
            .attr("height", "80px")
            .style("clip-path", "circle()")
            .on("mouseover", mouseover)
            .on("mousemove", mousemove)
            .on("mouseleave", mouseleave)
            .on("click", click);

        // Annotate links with optional "?" if the link is conjecture
        let linkLabel = graph.selectAll(".node")
            .data(root.descendants())
            .enter()
            .append("g")
            .attr("transform", d => `translate(${d.x},${d.y})`)
            .attr("class", "label");

        linkLabel.append("text")
            .attr("class", "label-text")
            .attr("dx", 12)
            .attr("dy", -4)
            .attr("text-anchor", "middle")
            .style("fill", "black")
            .style("stroke-width", "1px")
            .text(d => d.data.notes ? "?" : "");
    });
</script>

<div class="grid">
    <div id="graph-container" class="col-start-1 row-start-1 p-4 grid place-items-center">
        <svg id="graph"></svg>
    </div>

    <div class="col-start-1 row-start-1 p-4 w-1/5 h-max">
        <details class="bg-gray-300 duration-300 border-solid border-2 border-black pl-2 rounded-2xl">
            <summary class="bg-inherit px-5 py-3 text-lg font-bold cursor-pointer">
                Legend
            </summary>
            <div class="p-1">
                <div class="text-md font-bold">Style</div>
                <div class="text-md color-unryu">Unryū</div>
                <div class="text-md color-shiranui">Shiranui</div>
                <div class="text-md color-unknown">Unknown</div>
                <br>
                <div class="text-md font-bold">Notes</div>
                <div class="text-md"><strong>?</strong> represents an unconfirmed link</div>
                <br>
                <div class="text-md font-bold">Actions</div>
                <div class="text-md">Hover over rikishi to read more</div>
                <div class="text-md">Click to visit SumoDB profile</div>
            </div>
        </details>
    </div>
</div>

<style>
    .color-unryu {
        color: #af4f30;
    }

    .color-shiranui {
        color: #498e7b;
    }

    .color-unknown {
        color: #6d6d6d;
    }
</style>

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
        let tooltip = d3.select("#graph-container")
            .append("div")
            .style("position", "absolute")
            .style("opacity", 0)
            .attr("class", "tooltip")
            .style("background-color", "lightgray")
            .style("border", "solid")
            .style("border-width", "2px")
            .style("border-radius", "5px")
            .style("padding", "5px");

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
                .style("left", (event.offsetX + 235) + "px")
                .style("top", event.offsetY + "px");

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
            .attr("cy", "40")
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
            .attr("x", "-40px")
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
    <div class="p-4 col-start-1 row-start-1 drop-shadow-2xl -z-50">
        <div class="bg-gray-300 w-1/4 border-solid border-4 border-white pl-2 rounded-2xl">
            <div class="p-1">
                <table>
                    <thead>
                    <tr>
                        <td class="text-2xl font-bold">Key</td>
                    </tr>
                    </thead>
                    <tbody>
                    <tr>
                        <td class="text-lg font-bold color-unryu">Unryū</td>
                    </tr>
                    <tr>
                        <td class="text-lg font-bold color-shiranui">Shiranui</td>
                    </tr>
                    <tr>
                        <td class="text-lg font-bold color-unknown">Unknown</td>
                    </tr>
                    <tr>
                        <td><br></td>
                    </tr>
                    <tr>
                        <td class="text-lg"><strong>?</strong> on a link represents an unconfirmed link</td>
                    </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <div class="p-4 col-start-1 row-start-1 -z-50">
        <div class="text-right font-bold italic">
            <p>Hover over rikishi to see more info, click to visit SumoDB profile</p>
        </div>
    </div>

    <div id="graph-container" class="col-start-1 row-start-1 p-4 grid place-items-center">
        <svg id="graph"></svg>
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

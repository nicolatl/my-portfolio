<script>
    import {base} from '$app/paths';
    import * as d3 from "d3";
    import { onMount } from "svelte";
    import {
        computePosition,
        autoPlacement,
        offset,
    } from '@floating-ui/dom';
    import Bar from '$lib/StackedBar.svelte';
    import FileLines from '$lib/FileLines.svelte';
    import StackedBar from '../../lib/StackedBar.svelte';
    import Scrolly from "svelte-scrolly";


    let commitTooltip;

    let colorScale = d3.scaleOrdinal(d3.schemeTableau10);



    let data = [];
    let commits = [];
    let width = 1000, height = 600;
    
    let margin = {top: 10, right: 10, bottom: 30, left: 20};

    let usableArea = {
        top: margin.top,
        right: width - margin.right,
        bottom: height - margin.bottom,
        left: margin.left
    };
    usableArea.width = usableArea.right - usableArea.left;
    usableArea.height = usableArea.bottom - usableArea.top;
    let commitProgress = 100;
    const timeScale = d3.scaleTime([new Date(2025, 2, 1), new Date(2025, 3, 17)], [0, 100]);
    $: commitMaxTime = timeScale.invert(commitProgress);
    $: dateFilterLabel = new Date(commitMaxTime)
                     .toLocaleString("en", {dateStyle: "full", timeStyle: "short"});
    let raceProgress = 100;
    $: raceMaxTime = timeScale.invert(raceProgress);





    onMount(async () => {
        data = await d3.csv(`${base}/loc.csv`, row => ({
            ...row,
            line: Number(row.line), // or just +row.line
            depth: Number(row.depth),
            length: Number(row.length),
            date: new Date(row.date + "T00:00" + row.timezone),
            datetime: new Date(row.datetime)
        }));
        commits = d3.groups(data, d => d.commit).map(([commit, lines]) => {
            let first = lines[0];
            let {author, date, time, timezone, datetime} = first;
            let ret = {
                id: commit,
                url: "https://github.com/nicolatl/my-portfolio/" + commit,
                author, date, time, timezone, datetime,
                hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
                totalLines: lines.length
            };

            // Like ret.lines = lines
            // but non-enumerable so it doesn’t show up in JSON.stringify
            Object.defineProperty(ret, "lines", {
                value: lines,
                configurable: true,
                writable: true,
                enumerable: false,
            });

            return ret;
        });
        console.log(commits);



    });

    $: minDate = d3.min(commits.map(d => d.date));
    $: maxDate = d3.max(commits.map(d => d.date));
    $: maxDatePlusOne = new Date(commitMaxTime);
    $: maxDatePlusOne.setDate(maxDatePlusOne.getDate() + 1);

    $: filteredCommits = commits.filter(commit => commit.datetime <= commitMaxTime);
    $: filteredLines = data.filter(datum => datum.datetime <= commitMaxTime);
    $: raceFilteredLines = data.filter(datum => datum.datetime <= raceMaxTime);


    $: xScale = d3.scaleTime()
                .domain([minDate, maxDatePlusOne])
                .range([usableArea.left, usableArea.right])
                .nice();

    $: yScale = d3.scaleLinear()
                .domain([24, 0])
                .range([usableArea.bottom, usableArea.top]);
    let xAxis, yAxis, yAxisGridlines;
    

    $: {
        d3.select(xAxis).call(d3.axisBottom(xScale));
        d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
        d3.select(yAxisGridlines).call(
		d3.axisLeft(yScale)
            .tickFormat("")
            .tickSize(-usableArea.width)
        );
    }
    let hoveredIndex = -1;
    $: hoveredCommit = filteredCommits[hoveredIndex] ?? hoveredCommit ?? {};
    let cursor = {x: 0, y: 0};
    let tooltipPosition = {x: 0, y: 0};
    async function dotInteraction (index, evt) {
        let hoveredDot = evt.target;
        if (evt.type === "mouseenter") {
            hoveredIndex = index;
            cursor = {x: evt.x, y: evt.y};
            tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
                strategy: "fixed", // because we use position: fixed
                middleware: [
                    offset(5), // spacing from tooltip to dot
                    autoPlacement() // see https://floating-ui.com/docs/autoplacement
                ],
            });        }
        else if (evt.type === "mouseleave") {
            hoveredIndex = -1
        }
        else if (evt.type === "click") {
            let commit = commits[index]
            if (!clickedCommits.includes(commit)) {
                // Add the commit to the clickedCommits array
                clickedCommits = [...clickedCommits, commit];
            }
            else {
                    // Remove the commit from the array
                    clickedCommits = clickedCommits.filter(c => c !== commit);
            }
        }

    }
    let clickedCommits = [];

    $: allTypes = Array.from(new Set(data.map(d => d.type)));
    $: selectedLines = (filteredCommits.length > 0 ? filteredCommits : commits).flatMap(d => d.lines);
    $: selectedCounts = d3.rollup(
        selectedLines,
        v => v.length,
        d => d.type
    );
    $: languageBreakdown = allTypes.map(type => [type, selectedCounts.get(type) || 0]);







    
</script>
<h1>Meta</h1>
This page includes stats about the code of this website.

<h3>Commits by time of day</h3>
<!-- <div class="slider-container">
    <label>
        Show commits before
        <input id="maxTimeSlider" type="range" min="0" max="100" bind:value={commitProgress} />
        <time style="display: block">
            {dateFilterLabel}
        </time>
    </label>
</div> -->
<dl class="stats">
	<dt>Total <abbr title="Lines of code">LOC</abbr></dt>
	<dd>{filteredLines.length}</dd>
	<dt>Total Commits</dt>
	<dd>{filteredCommits.length}</dd>
	<dt>Longest Line</dt>
	<dd>{d3.max(filteredLines,(d) => d.length)}</dd>
</dl>
<Scrolly bind:progress={ commitProgress }>
	{#each commits as commit, index }
        <p>
            On {commit.datetime.toLocaleString("en", {dateStyle: "full", timeStyle: "short"})},
            {index === 0 
                ? "I set forth on my very first commit, beginning a magical journey of code. You can view it "
                : "I added another enchanted commit, each line sparkling with a touch of wonder. See it "}
            <a href="{commit.url}" target="_blank">
                {index === 0 ? "here" : "here"}
            </a>.
            This update transformed {commit.totalLines} lines across { d3.rollups(commit.lines, D => D.length, d => d.file).length } files.
            With every commit, our project grows into a kingdom of dreams.
        </p>
    {/each}

	<svelte:fragment slot="viz">
		<svg viewBox="0 0 {1*width} {height}">
            <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
            <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
            <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
            <g class="dots">
                {#each filteredCommits as commit, index (commit.id)  }
                    <circle 
                        class:selected={ clickedCommits.includes(commit) }
                        on:click={ evt => dotInteraction(index, evt) }
                        on:mouseenter={evt => dotInteraction(index, evt)}
                        on:mouseleave={evt => dotInteraction(index, evt)}
                        cx={ xScale(commit.datetime) }
                        cy={ yScale(commit.hourFrac) }
                        r="5"
                        fill="steelblue"
                    />
                {/each}
            </g>  
        </svg>
        <StackedBar data={languageBreakdown} width={0.67*width} colorScale={colorScale} />
	</svelte:fragment>
</Scrolly>
<Scrolly bind:progress={ raceProgress }  --scrolly-layout="viz-first" --scrolly-viz-width="3fr">
	{#each commits as commit, index }
        <p>
            On {commit.datetime.toLocaleString("en", {dateStyle: "full", timeStyle: "short"})},
            {index === 0 
                ? "I set forth on my very first commit, beginning a magical journey of code. You can view it "
                : "I added another enchanted commit, each line sparkling with a touch of wonder. See it "}
            <a href="{commit.url}" target="_blank">
                {index === 0 ? "here" : "here"}
            </a>.
            This update transformed {commit.totalLines} lines across { d3.rollups(commit.lines, D => D.length, d => d.file).length } files.
            With every commit, our project grows into a kingdom of dreams.
        </p>
    {/each}
	<svelte:fragment slot="viz">
		<FileLines lines={raceFilteredLines} width={0.6*width} colorScale={colorScale} svgWidth={0.6*width}/>
	</svelte:fragment>
</Scrolly>




<dl class="info tooltip" hidden={hoveredIndex === -1} style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px" bind:this={commitTooltip}>
    <dt>Commit</dt>
    <dd><a href="{ hoveredCommit.url }" target="_blank">{ hoveredCommit.id }</a></dd>

    <dt>Date</dt>
    <dd>{ hoveredCommit.datetime?.toLocaleString("en", {dateStyle: "full"}) }</dd>

    <!-- TODO Add: Time, author, lines edited -->
</dl>






<style>

    :global(body) {
        max-width: min(120ch, 80vw);
    }

    .selected {
        fill: var(--color-accent);
    }

    dl.info {
        display: grid;
        grid-template-columns: max-content 1fr;
        margin:0;
        transition-duration: 500ms;
        transition-property: opacity, visibility;

        &[hidden]:not(:hover, :focus-within) {
            opacity: 0;
            visibility: hidden;
        }
    }
    dl.info dt {
        grid-column: 1;
        grid-row: auto;
    }
    
    dl.info dd {
        grid-column: 2;
        grid-row: auto;
    }
    .tooltip{
        position:fixed;
        top:1em;
        left:1em;
        background-color: white;
        box-shadow: 2px 2px 2px grey, -2px -2px 2px grey;
        border-radius:1em;
        /* width: 3em; */
    }
    circle {
        transition: 200ms;
        @starting-style {
            r: 0;
        }


        &:hover {
            transform: scale(1.5);
        }
        transform-origin: center;
        transform-box: fill-box;

    }

    svg {
		overflow: visible;
	}
    dl {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        padding: 20px;
        box-shadow: 2px 2px 2px grey, -2px -2px 2px grey;
    }
    dt {
        grid-row: 1;
        text-align: center;
    }
    dd {
        grid-row: 2;
        padding-left: 0;
        text-align: center;
    }
    .gridlines {
        stroke-opacity: .2;
    }

</style>


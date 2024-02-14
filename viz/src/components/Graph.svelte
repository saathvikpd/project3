<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    let chartWidth = 800;
    let chartHeight = 800;

    let year1 = 0;
    let year2 = 0;

    const paddings = {
                top: 50,
                left: 50,
                right: 50,
                bottom: 50,
            };

    let nr = ['coal_share_elec', 'nuclear_share_elec', 'gas_share_elec', 'oil_share_elec'];
    let r = ['solar_share_elec', 'biofuel_share_elec', 'other_renewables_share_elec_exc_biofuel', 'wind_share_elec', 'hydro_share_elec'];
    const all_cols = r.concat(nr);

    let data = [];
    let subset = [];

    let xScale = d3.scaleLinear().domain(d3.extent(subset, (d) => d[nr[0]])).range([paddings.left, chartWidth - paddings.right]);
    let yScale = d3.scaleLinear().domain(d3.extent(subset, (d) => d[nr[1]])).range([paddings.top, chartHeight - paddings.bottom]);

    let cumul = [];

    let gx;
    let gy;

    let hovered = -1; 

	let recorded_mouse_position = {
		x: 0, y: 0
	};

    let areas = [];
    
    let area1 = d3.area()
            .x(function(d) { return xScale(d[0]); })
            .y0(chartHeight - paddings.bottom)
            .y1(function(d) { return yScale(d[5]); });

    let area2 = d3.area()
            .x(function(d) { return xScale(d[0]); })
            .y0(function(d) { return yScale(d[5]); })
            .y1(yScale(100));


    let colorScale1 = d3.scaleLinear().domain([0, 4.5, all_cols.length - 1]).range(["blue", "white", "red"]);
    let colorScale2 = d3.scaleLinear().domain([0, 4.5, all_cols.length - 1]).range(["lightsteelblue", "white", "pink"]);

    onMount(async () => {

        const res = await fetch('owid-energy-data.csv'); 

        const csv = await res.text();

        data = d3.csvParse(csv, d3.autoType)

    });

    
    
    
    $: {

        if (data.length > 0) {
            let country = "United States";
            year1 = 2000;
            year2 = 2023;

            let item;
            let sub_item;

            let temp = all_cols;

            let running_sum = 0;
            for (let i = 0; i < data.length; ++i) {
                item = data[i];
                if (item["country"] === country && item["year"] >= year1 && item["year"] < year2) {
                    sub_item = [item["year"]];
                    running_sum = 0;
                    for (let j = 0; j < temp.length; ++j) {
                        running_sum = running_sum + item[temp[j]];
                        sub_item.push(running_sum);
                        
                    }
                    subset.push(sub_item);
                }
            }

            

            
            
            xScale = d3.scaleLinear().domain([year1, year2]).range([paddings.left, chartWidth - paddings.right]);
            yScale = d3.scaleLinear().domain([100, 0]).range([paddings.top, chartHeight - paddings.bottom]);

            area1 = d3.area()
            .x(function(d) { return xScale(d[0]); })
            .y0(chartHeight - paddings.bottom)
            .y1(function(d) { return yScale(d[5]); });

            area2 = d3.area()
            .x(function(d) { return xScale(d[0]); })
            .y0(function(d) { return yScale(d[5]); })
            .y1(yScale(100));

            let temp_area;
            for (let i = 0; i < all_cols.length; ++i) {
                if (i === 0) {
                    temp_area = d3.area()
                    .x(function(d) { return xScale(d[0]); })
                    .y0(chartHeight - paddings.bottom)
                    .y1(function(d) { return yScale(d[1]); });
                }
                else {
                    temp_area = d3.area()
                    .x(function(d) { return xScale(d[0]); })
                    .y0(function(d) { return yScale(d[i]); })
                    .y1(function(d) { return yScale(d[i + 1]); });
                }
                areas.push(temp_area);
            }

            d3.select(gy).call(d3.axisLeft(yScale).ticks(null))
            d3.select(gx).call(d3.axisBottom(xScale).ticks(10, "f"))
        }
    
    
        

        
    }

    
    

</script>

<h1>Where have we been getting our electricity from in the 21st century?</h1>
<svg width={chartWidth} height={chartHeight}>
    
    <g>
        <path
            d = {area1(subset)}
            fill = {hovered === 1 ? "blue": "lightsteelblue"}
            on:mouseover={(event) => { hovered = 1; 
				recorded_mouse_position = {
							x: event.pageX,
							y: event.pageY
						}

				}}
            on:mouseout={(event) => { hovered = -1; }}
        >
    </g>

    <g>
        <path
            d = {area2(subset)}
            fill = {hovered === 0 ? "red": "pink"}
            on:mouseover={(event) => { hovered = 0; 
				recorded_mouse_position = {
							x: event.pageX,
							y: event.pageY
						}

				}}
            on:mouseout={(event) => { hovered = -1; }}
        >
    </g>


    
    {#each areas as area, index}
    <g>
        <path
            d = {area(subset)}
            fill = {index === hovered ? colorScale1(index): colorScale2(index)}
            on:mouseover={(event) => { hovered = index; 
                recorded_mouse_position = {
                            x: event.pageX,
                            y: event.pageY
                        }

                }}
            on:mouseout={(event) => { hovered = -1; }}
        >
    </g>
    {/each}

    <g bind:this={gy} transform="translate({paddings.left} , 0)" />
    <g bind:this={gx} transform="translate(0, {chartHeight - paddings.bottom})" />


    <g>
    {#each subset as row, index}
    {#if index < subset.length - 1}
        {#each row as col, index1}
            {#if index1 > 0 && index1 <= 5}
                <line
                    x1={xScale(row[0])}
                    x2={xScale(subset[index + 1][0])}
                    y1={yScale(col)}
                    y2={yScale(subset[index + 1][index1])}
                    stroke="blue"
                    stroke-width="2"
                />
            {/if}
            {#if index1 > 5}
                <line
                    x1={xScale(row[0])}
                    x2={xScale(subset[index + 1][0])}
                    y1={yScale(col)}
                    y2={yScale(subset[index + 1][index1])}
                    stroke="red"
                    stroke-width="2"
                />
            {/if}
        {/each}
    {/if}
    {/each}
    </g>
</svg>





<style>
    main {
    }

    h1 {
        height: 0em;
        font-size: 25px;
        font-weight: 300;
        line-height: 0.5;
        text-align: center
    }

</style>

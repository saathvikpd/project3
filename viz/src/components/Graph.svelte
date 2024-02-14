<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    let chartWidth = 700;
    let chartHeight = 500;

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

    onMount(async () => {

        const res = await fetch('owid-energy-data.csv'); 

        const csv = await res.text();

        data = d3.csvParse(csv, d3.autoType)

    });

    
    
    
    $: {

        if (data.length > 0) {
            let country = "United States";
            let year1 = 2000;
            let year2 = 2023;

            let item;
            let sub_item;

            let temp = all_cols;

            let running_sum = 0;
            for (let i = 0; i < data.length; ++i) {
                item = data[i];
                if (item["country"] == country && item["year"] >= year1 && item["year"] < year2) {
                    sub_item = [item["year"]];
                    running_sum = 0;
                    for (let j = 0; j < temp.length; ++j) {
                        running_sum = running_sum + item[temp[j]];
                        sub_item.push(running_sum);
                        
                    }
                    subset.push(sub_item);
                }
            }

            console.log(subset);

            

            
            
            xScale = d3.scaleLinear().domain([year1, year2]).range([paddings.left, chartWidth - paddings.right]);
            yScale = d3.scaleLinear().domain([100, 0]).range([paddings.top, chartHeight - paddings.bottom]);


        }
        

        
    }

    
    

</script>

<svg width={chartWidth} height={chartHeight}>
 <g>
   <line
     x1={paddings.left}
     x2={chartWidth - paddings.right}
     y1={chartHeight - paddings.bottom}
     y2={chartHeight - paddings.bottom}
     stroke="black"
     stroke-width="2"
   />
   <line
     x1={paddings.left}
     x2={paddings.left}
     y1={paddings.top}
     y2={chartHeight - paddings.bottom}
     stroke="black"
     stroke-width="2"
   />

   <g>
   {#each subset as row, index}
   {#if index < subset.length - 1}
        {#each row as col, index1}
            {#if index1 > 0 && index1 <= 4}
                <line
                    x1={xScale(row[0])}
                    x2={xScale(subset[index + 1][0])}
                    y1={yScale(col)}
                    y2={yScale(subset[index + 1][index1])}
                    stroke="black"
                    stroke-width="1"
                />
            {/if}
            {#if index1 > 4}
                <line
                    x1={xScale(row[0])}
                    x2={xScale(subset[index + 1][0])}
                    y1={yScale(col)}
                    y2={yScale(subset[index + 1][index1])}
                    stroke="red"
                    stroke-width="1"
                />
            {/if}
        {/each}
   {/if}
   {/each}
   </g>
   



   
  </g>
</svg>





<style>
    main {
    }

    h1 {
        font-size: 50px;
        font-weight: 300;
        line-height: 0.5;
        text-align: center
    }

</style>

<script>
    import * as d3 from 'd3';
    let chartWidth = 500;
    let chartHeight = 300;

    

    const paddings = {
                top: 50,
                left: 50,
                right: 50,
                bottom: 50,
            };

    let nr = ['nuclear_share_elec', 'coal_share_elec', 'gas_share_elec', 'oil_share_elec'];
    let r = ['solar_share_elec', 'biofuel_share_elec', 'other_renewables_share_elec_exc_biofuel', 'wind_share_elec', 'hydro_share_elec'];

    let url = "https://raw.githubusercontent.com/owid/energy-data/master/owid-energy-data.csv";

    let data = [];
    d3.csv(url).then(function (d) {
        data = d
    })

    let subset = [];

    let xScale = d3.scaleLinear().domain(d3.extent(subset, (d) => d[nr[0]])).range([paddings.left, chartWidth - paddings.right]);
    let yScale = d3.scaleLinear().domain(d3.extent(subset, (d) => d[nr[1]])).range([paddings.top, chartHeight - paddings.bottom]);

    $: {
        if (data.length !== 0) {
            
            let country = "United States";
            let year = 2000;

            let item;
            let sub_item;

            let temp = nr;
            temp.unshift("year");

            for (let i = 0; i < data.length; ++i) {
                item = data[i];
                sub_item = {};
                if (item["country"] == country && item["year"] >= year) {
                    for (let j = 0; j < temp.length; ++j) {
                        sub_item[temp[j]] = item[temp[j]];
                    }
                    subset.push(sub_item);
                }
            }

            
            
            
            xScale = d3.scaleLinear().domain(d3.extent(subset, (d) => d[nr[0]])).range([paddings.left, chartWidth - paddings.right]);
            yScale = d3.scaleLinear().domain(d3.extent(subset, (d) => d[nr[1]])).range([paddings.top, chartHeight - paddings.bottom]);

            


        }

        
    }

    console.log(xScale(2005));
    

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
        <line
            x1={xScale(row[nr[0]])}
            x2={xScale(subset[index + 1][nr[0]])}
            y1={yScale(row[nr[1]])}
            y2={yScale(subset[index + 1][nr[1]])}
            stroke="black"
            stroke-width="2"
        />
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

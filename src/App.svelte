<script>
    import { onMount } from 'svelte';
    import InputField from './components/InputField.svelte';
    import Plotly from 'plotly.js-dist-min';
    import { Client, Functions } from 'appwrite';

    let ProfitOverTimeChart;
    let profitOverTimeLayout;

    // inputs
    let entryTime = "";
    let spreadWidth = "";
    let entryCredit = "";
    let numberOfSpreads = "";
    let stopPrice = "";
    let limitPrice = "";
    let stopLossMultiplier = "";

    // sets graph height to match input container height
    onMount(() => {
      const input = document.getElementById('container-input');
      const graph = document.getElementById('container-graph');
      graph.style.height = `${input.offsetHeight}px`;
    });

    // initalize plotly chart
    onMount(() => {
      profitOverTimeLayout = {
        title: {
          text: 'Profit Over Time',
          font: {
            color: 'white',
            size: 15
          },
          x: 0.5
        },
        margin: {
          t: 25,
          b: 50,
          l: 60,
          r: 25
        },
        autosize: true,
        xaxis: {
          title: {
            text: 'Date',
            font: {
              size: 15,
              color: 'white'
            }
          },
          tickfont: {
            color: '#C9C9CA',
            size: 12
          },
          linecolor: '#C9C9CA',
          type: 'date',
          tickformat: '%b %d',
          range: ['2024-02-12', '2024-02-16'],
          dtick: 86400000,
        },
        yaxis: {
          title: {
            text: 'Profit',
            font: {
              size: 15,
              color: 'white'
            },
            standoff: 5,
          },
          automargin: true,
          tickfont: {
            color: '#C9C9CA',
            size: 12
          },
          linecolor: '#C9C9CA',
          range: [0, 100],
        },
        showlegend: false,
        paper_bgcolor: '#161616',
        plot_bgcolor: '#161616',
      };
      
      Plotly.newPlot(ProfitOverTimeChart, [{x: [], y: [], type: 'scatter'}], profitOverTimeLayout, { responsive: true });
      Plotly.Plots.resize(ProfitOverTimeChart);
    });

    // plot graph
    function plotProfitOverTime(dates, profitOverTime) {
      // format to yyyy-mm-dd
      for (let i=0; i<dates.length; i++) {
        const day = dates[i].slice(6,8);
        const month = dates[i].slice(4,6);
        const year = dates[i].slice(0,4);
        dates[i] = `${year}-${month}-${day}`;
      }

      // adjust y-axis range based on data
      const max = (Math.max(...profitOverTime) / 10) * 10;
      const min = (Math.min(...profitOverTime) / 10) * 10;
      if (min >= 0) {
        profitOverTimeLayout.yaxis.range = [0, max + 100];
      } else if (max <= 0) {
        profitOverTimeLayout.yaxis.range = [min - 100, 0];
      } else {
        profitOverTimeLayout.yaxis.range = [min - 100, max + 100];
      }

      // interpolate zero crossings
      let newDates = [];
      let newProfits = [];
      let fakeReal = [];
      for (let i=1; i<profitOverTime.length; i++) {
        let prevDate = dates[i-1];
        let prevPrice = profitOverTime[i-1];
        let curDate = dates[i];
        let curPrice = profitOverTime[i];

        // add first point
        if (i === 1) {
          newDates.push(prevDate);
          newProfits.push(prevPrice);
          fakeReal.push('real');
        }

        // detect sign change
        if (Math.sign(prevPrice) !== Math.sign(curPrice)) {
          //add time halfway between prev date and cur date
          let midDate = new Date((new Date(prevDate).getTime() + new Date(curDate).getTime()) / 2). toISOString().replace('.000Z', '');
          newDates.push(midDate);
          newProfits.push(0);
          fakeReal.push('fake');
        }

        // add current point
        newDates.push(curDate);
        newProfits.push(curPrice);
        fakeReal.push('real');
      }

      const positiveTrace = {
        x: newDates,
        y: newProfits.map(val => val >= 0 ? val : null),
        type: 'scatter',
        mode: 'lines',
        line: { color: '#339D01', width: 3 },
        fill: 'tozeroy',
        fillcolor: 'rgba(51, 157, 1, 0.2)',
        hoverinfo: 'skip'
      };

      const negativeTrace = {
        x: newDates,
        y: newProfits.map(val => val <= 0 ? val : null),
        type: 'scatter',
        mode: 'lines',
        line: { color: '#DF2828', width: 3 },
        fill: 'tozeroy',
        fillcolor: 'rgba(223, 40, 40, 0.2)',
        hoverinfo: 'skip'
      };

      const originalTrace = {
        x: dates,
        y: profitOverTime,
        type: 'scatter',
        mode: 'markers',
        marker: { color: 'white', size: 5 },
        hovertemplate: '<b>Date:</b> %{x}<br><b>Profit:</b> %{y}<extra></extra>'
      };
      
      // update plotly chart
      Plotly.newPlot(ProfitOverTimeChart, [positiveTrace, negativeTrace, originalTrace], profitOverTimeLayout, { responsive: true });
      Plotly.Plots.resize(ProfitOverTimeChart);
    }

    // appwrite client setup
    const client = new Client()
    .setEndpoint('https://tor.cloud.appwrite.io/v1')
    .setProject('690eb041002cafd8d0c7');
    const functions = new Functions(client);

    async function getResults() {
      const parameters = {
        entryTime,
        spreadWidth,
        entryCredit,
        numberOfSpreads,
        stopPrice,
        limitPrice,
        stopLossMultiplier
      };
      console.log("data to send:", parameters);

      try {
        // call appwrite function
        const execution = await functions.createExecution(
          '690eb25c0005266fa532',
          JSON.stringify(parameters),
          false // false = synchronous (wait for response), true = asynchronous (do not wait)
        );

        // parse response
        try {
          let functionResponse = JSON.parse(execution.responseBody).response;

          const totalProfit = functionResponse.totalProfit;
          const dates = functionResponse.dates;
          const profitOverTime = functionResponse.profitOverTime;

          plotProfitOverTime(dates, profitOverTime);
        } catch (error) {
            console.error('failed to parse response:', error);
        }
      } catch (error) {
        console.error('error:', error);
      }
    }
</script>

<main>
  <div class="top-nav-bar">
    <h1 id="top-nav-bar-title">Backtester</h1>
  </div>

  <div class="container-horizontal">
    <!-- input parameters -->
    <div class="container-child" id="container-input">
        <h2>Configuration</h2>

        <div class="container-horizontal">
          <InputField Label="Entry Time" bind:Value={entryTime}/>
          <InputField Label="Spread Width" bind:Value={spreadWidth}/>
        </div>

        <div class="container-horizontal">
          <InputField Label="Entry Credit" bind:Value={entryCredit}/>
          <InputField Label="Number of Spreads" bind:Value={numberOfSpreads}/>
        </div>

        <div class="container-horizontal">
          <InputField Label="Stop Price" bind:Value={stopPrice}/>
          <InputField Label="Limit Price" bind:Value={limitPrice}/>
        </div>

        <div class="container-horizontal">
          <InputField Label="Stop Loss Multiplier" bind:Value={stopLossMultiplier}/>
          <div id="BacktestButtonContainer">
            <button id="BacktestButton" type="button" on:click={getResults}>Run Backtest</button>
          </div>
        </div>
    </div>

    <!-- graph -->
    <div class="container-child" id="container-graph">
      <div id="ProfitOverTimeChart" bind:this={ProfitOverTimeChart}></div>
    </div>
  </div>
</main>

<style>
  h1 {
    font-weight: bold;
    font-size: 2.5rem;
  }

  h2 {
    font-weight: bold;
    font-size: 1.25rem;
  }

  .top-nav-bar {
    background-color: rgb(0, 0, 0);
    color: rgb(255, 255, 255);
    font-weight: bold;
    font-size: 2.5rem;
  }

  #top-nav-bar-title {
    margin-left: 2rem;
  }

  .container-horizontal {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: flex;
    gap: 1rem;
    margin: 1rem;
  }

  .container-child {
    background-color: #161616;
    padding: 1rem;
  }

  #container-input {
    width: 50%;
  }

  #container-graph {
    width: 50%;
    overflow: hidden;
  }
  
  #ProfitOverTimeChart {
    width: 100%;
    height: 100%;
  }

  #BacktestButtonContainer {
    width: 100%;
    display: flex;
    flex-direction: column;
    justify-content: flex-end; 
  }

  #BacktestButton {
    width: 100%;
    height: 1.75rem;
    background-color: #0C40C2;
    border: none;
    text-align: center;
    color: white;
    cursor: pointer;
  }

  #BacktestButton:hover {
    background-color: #0A30A0;
  }

  #BacktestButton:active {
    background-color: #082080;
  }

</style>

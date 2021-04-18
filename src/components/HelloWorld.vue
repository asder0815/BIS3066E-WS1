<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-btn elevation="2" @click="cleanData" :disabled=dataCleaned>Clean Data</v-btn>
        <v-checkbox v-model="fullReport" :label="`Full report: ${fullReport.toString()}`" :disabled=dataCleaned></v-checkbox>
        <v-checkbox v-model="onlyBlockbusters" :label="`Only evaluate blockbusters: ${onlyBlockbusters.toString()}`" :disabled=dataCleaned></v-checkbox>
      </v-col>  
    </v-row>
    <v-btn elevation="2" @click="analyzeTextAtrribute(['actor_1_name','actor_2_name','actor_3_name'], 'actorValueChart')" :disabled=!dataCleaned>Actor Values</v-btn>
    <v-btn elevation="2" @click="analyzeTextAtrribute(['director_name'], 'directorValueChart')" :disabled=!dataCleaned>Director Values</v-btn>
    <v-btn elevation="2" @click="analyzeKeywords('genres', 'genreValueChart')" :disabled=!dataCleaned>Genres</v-btn>
    <v-btn elevation="2" @click="analyzeKeywords('plot_keywords', 'plotValueChart')" :disabled=!dataCleaned>Plot</v-btn>
    <v-btn elevation="2" @click="analyzeKeywordProfit('genres', 'genreProfitChart')" :disabled=!dataCleaned>Genres Profit</v-btn>
    <v-btn elevation="2" @click="analyzeKeywordProfit('plot_keywords', 'plotProfitChart')" :disabled=!dataCleaned>Plot Profit</v-btn>
    <v-btn elevation="2" @click="analyzeDiminishingReturns()" :disabled=!dataCleaned>Update Graph</v-btn>

    <v-sparkline
      :value="graphValue"
      :gradient="graphData.gradient"
      :smooth="graphData.radius || false"
      :padding="graphData.padding"
      :line-width="graphData.width"
      :stroke-linecap="graphData.lineCap"
      :gradient-direction="graphData.gradientDirection"
      :fill="graphData.fill"
      :type="graphData.type"
      :auto-line-width="graphData.autoLineWidth"
      auto-draw
    ></v-sparkline>

    <apexchart ref="actorValueChart" type="bar" height="350" :options="actorValueChartOptions" :series="actorValueChartData"></apexchart>
    <apexchart ref="directorValueChart" type="bar" height="350" :options="directorValueChartOptions" :series="directorValueChartData"></apexchart>
    <apexchart ref="genreValueChart" type="bar" height="350" :options="genreValueChartOptions" :series="genreValueChartData"></apexchart>
    <apexchart ref="plotValueChart" type="bar" height="350" :options="plotValueChartOptions" :series="plotValueChartData"></apexchart>
    <apexchart ref="genreProfitChart" type="bar" height="350" :options="genreProfitChartOptions" :series="genreProfitChartData"></apexchart>
    <apexchart ref="plotProfitChart" type="bar" height="350" :options="plotProfitChartOptions" :series="plotProfitChartData"></apexchart>

  </v-container>
</template>

<script>
  import Vue from 'vue'; 
  import VueApexCharts from 'vue-apexcharts'; 
  Vue.component('apexchart', VueApexCharts); 
  const gradients = [['#222'],['#42b3f4'],['red', 'orange', 'yellow'],['purple', 'violet'],['#00c6ff', '#F0F', '#FF0'],['#f72047', '#ffd200', '#1feaea']]; 
  export default {
    name: 'HelloWorld',
    props: {
      movieData: Array
    },
    data: () => ({
      cleanedData: [], 
      dataCleaned: false,
      blockbusters: [], 
      fullReport: false, 
      onlyBlockbusters: true, 
      blockbusterGross: 100000000, 
      maxBudget: 350000000,
      maxGrossing: 800000000,
      maxProfitMargin: 50, 
      minProfitMargin: 0.01,
      profitMarginTarget: 2, 
      budgetInterval: 25000000,
      showMaxChart: 20, 
      maxDecimals: 2, 
      graphValue: [0, 2, 5, 9, 5, 10, 3, 5, 0, 0, 1, 8, 2, 9, 0], 
      graphData: {
        width: 2,
        radius: 10,
        padding: 8,
        lineCap: 'round',
        gradient: gradients[5],
        gradientDirection: 'top',
        gradients,
        fill: false,
        type: 'trend',
        autoLineWidth: false
      }, 
      actorValueChartData: [{ data: [] }],
      actorValueChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Actor Value' },
        xaxis: { categories: [] }
      }, 
      directorValueChartData: [{ data: [] }],
      directorValueChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Director Value' },
        xaxis: { categories: [] }
      }, 
      genreValueChartData: [{ data: [] }],
      genreValueChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Genre Value' },
        xaxis: { categories: [] }
      }, 
      plotValueChartData: [{ data: [] }],
      plotValueChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Plot Keyword Value' },
        xaxis: { categories: [] }
      }, 
      genreProfitChartData: [{ data: [] }],
      genreProfitChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Genre Profit Chance' },
        xaxis: { categories: [] }
      }, 
      plotProfitChartData: [{ data: [] }],
      plotProfitChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Plot Keyword Profit Chance' },
        xaxis: { categories: [] }
      }
    }),
    methods: {
      cleanData() {
        var loopCounter = 0; 
        var filteredCounter = 0; 
        var cleanStepOne = []; 
        this.movieData.forEach(movie => {
          if(movie.name == "") {
            if(this.fullReport) console.log("Filtered movie " + loopCounter + " due to no name."); 
            filteredCounter++; 
          } else if(movie.gross == "") {
            if(this.fullReport) console.log("Filtered movie: " + movie.movie_title + " due to no grossing data.");
            filteredCounter++;  
          } else if(movie.budget == "") {
            if(this.fullReport) console.log("Filtered movie: " + movie.movie_title + " due to no budget data.");
            filteredCounter++;  
          }
          else {
            cleanStepOne.push(movie); 
          }
          loopCounter++; 
        }); 
        console.log("--- Finished step two of cleaning data ---");
        console.log("Filtered out " + filteredCounter + " movies due to missing data.");
        filteredCounter = 0; 
        var cleanStepTwo = []; 
        var linkSet = new Set(); 
        cleanStepOne.forEach(movie => {
          if(movie.movie_imdb_link != "" && linkSet.size === linkSet.add(movie.movie_imdb_link).size) {
            if(this.fullReport) console.log("Filtered out movie " + movie.movie_title + " due to duplicate IMBD link."); 
            filteredCounter++; 
          }
          else if(movie.movie_title != "" && linkSet.size === linkSet.add(movie.movie_title).size) {
            if(this.fullReport) console.log("Filtered out movie " + movie.movie_title + " due to duplicate title."); 
            filteredCounter++;
          }
          else {
            cleanStepTwo.push(movie); 
          }
        });
        console.log("--- Finished step two of cleaning data ---");
        console.log("Filtered out " + filteredCounter + " movies due to duplicate data.");  
        var filteredCounterProfit = 0; 
        var filteredCounterBudget = 0; 
        var filteredCounterProfitMargin = 0; 
        var cleanStepThree = []; 
        cleanStepTwo.forEach(movie => {
          if(movie.gross > this.maxGrossing) {
            if(this.fullReport) console.log("Filtered out movie " + movie.movie_title + " due to unrealistic grossing."); 
            filteredCounterProfit++;
          } else if(movie.budget > this.maxBudget) {
            if(this.fullReport) console.log("Filtered out movie " + movie.movie_title + " due to unrealistic budget."); 
            filteredCounterBudget++;
          } else if(movie.gross / movie.budget > this.maxProfitMargin || movie.gross / movie.budget < this.minProfitMargin) {
            if(this.fullReport) console.log("Filtered out movie " + movie.movie_title + " due to unrealistic profit margins."); 
            filteredCounterProfitMargin++;
          } else {
            cleanStepThree.push(movie); 
          }
        }); 
        console.log("--- Finished step three of cleaning data ---");
        console.log("Filtered out " + filteredCounterProfit + " movies due to unrealistic profit numbers.");
        console.log("Filtered out " + filteredCounterBudget + " movies due to unrealistic budget numbers.");
        console.log("Filtered out " + filteredCounterProfitMargin + " movies due to unrealistic profit margins.");
        console.log("--- Finished cleaning data ---")
        this.cleanedData = this.onlyBlockbusters ? this.createBlockbusterList(cleanStepTwo) : cleanStepTwo; 
        this.dataCleaned = true; 
      }, 
      createBlockbusterList(movieList) {
        var result = []; 
        movieList.forEach(movie => {
          if(movie.gross > this.blockbusterGross) {
            result.push(movie); 
          }
        });
        return result; 
      },
      analyzeTextAtrribute(attributeList, chartName, normalize = true) {
        var result = []; 
        var baseGross = this.onlyBlockbusters ? this.blockbusterGross : 1; 
        this.cleanedData.forEach(movie => {
          var valueList = []; 
          attributeList.forEach(valueNr => {
            if(movie[valueNr] != "") valueList.push({name: movie[valueNr], value: movie.gross / baseGross}); 
          }); 
          valueList.forEach(value => {
            if(result.some(e => e.name === value.name)) result[result.findIndex(e => e.name === value.name)].value += value.value; 
            else result.push(value); 
          });
        }); 
        this.updateValueGraph(result, normalize, chartName); 
      }, 
      analyzeKeywords(attribute, chartName, normalize = true) {
        var result = []; 
        var baseGross = this.onlyBlockbusters ? this.blockbusterGross : 1; 
        this.cleanedData.forEach(movie => {
          var keywordList = []; 
          if(movie[attribute] != "") {
            movie[attribute].split("|").forEach(keyword => {
              keywordList.push({name: keyword, value: movie.gross / baseGross}); 
            }); 
          }
          keywordList.forEach(keyword => {
            if(result.some(e => e.name === keyword.name)) result[result.findIndex(e => e.name === keyword.name)].value += keyword.value; 
            else result.push(keyword); 
          });
        }); 
        this.updateValueGraph(result, normalize, chartName); 
      }, 
      analyzeKeywordProfit(attribute, chartName) {
        var result = []; 
        this.cleanedData.forEach(movie => {
          var keywordList = []; 
          if(movie[attribute] != "") {
            movie[attribute].split("|").forEach(keyword => {
              keywordList.push({
                name: keyword, 
                above: (movie.gross / movie.budget) > this.profitMarginTarget ? 1 : 0, 
                below: (movie.gross / movie.budget) > this.profitMarginTarget ? 0 : 1, 
                value: 0}); 
            }); 
          }
          keywordList.forEach(keyword => {
            if(result.some(e => e.name === keyword.name)) {
              var target = result[result.findIndex(e => e.name === keyword.name)]; 
              target.above += keyword.above;
              target.below += keyword.below; 
              target.value = (target.above) / (target.above + target.below); 
            }
            else {
              keyword.value = (keyword.above + keyword.below) / (keyword.above); 
              result.push(keyword); 
            }
          });
        }); 
        result.forEach(keyword => {
          if(keyword.value == Infinity) keyword.value = 0; 
          keyword.value = (keyword.value * 100).toFixed(this.maxDecimals); 
          console.log(keyword.name + " has a " + keyword.value + "% chance to exceed a profit margin of " + this.profitMarginTarget + "with " + keyword.above + " values above and " + keyword.below + " values below"); 
        }); 
        this.updateValueGraph(result, false, chartName); 
      },
      analyzeDiminishingReturns() {
        var result = []; 
        this.cleanedData.forEach(movie => {
          if(result.some(e => e.interval === Math.floor(movie.budget / this.budgetInterval))) {
            var target = result[result.findIndex(e => e.interval === Math.floor(movie.budget / this.budgetInterval))]; 
            target.amount++; 
            target.sum += movie.gross; 
            target.average = target.sum / target.amount; 
          }
          else {
            result.push({
              interval: Math.floor(movie.budget / this.budgetInterval), 
              amount: 1, 
              sum: movie.gross, 
              average: movie.gross
            }); 
          }
        }); 
        console.log(result);
        var graphValues = []; 
        result.sort(this.sortByInterval).forEach(res => {
          if(res.interval <= Math.floor(this.maxBudget / this.budgetInterval)) graphValues.push(res.average); 
        }); 
        console.log(graphValues); 
        this.graphValue = graphValues; 
      },
      sortByValue(a,b) {
        return a.value === b.value ? 0 : a.value > b.value ? 1 : -1; 
      },
      sortByInterval(a,b) {
        return a.interval === b.interval ? 0 : a.interval > b.interval ? 1 : -1; 
      },
      normalizeByValue(array) {
        array.sort(this.sortByValue); 
        var max = array[array.length-1].value; 
        array.forEach(element => {
          element.value = (element.value * 100 / max).toFixed(this.maxDecimals); 
        }); 
        return array; 
      }, 
      updateValueGraph(result, normalize, chartName) {
        result = this.parseValues(result); 
        if(normalize) result = this.normalizeByValue(result); 
        else result.sort(this.sortByValue); 
        var dataHandle = chartName + "Data"; 
        var optionHandle = chartName + "Options";
        var newData = [{data: []}]; 
        var newOptions = JSON.parse(JSON.stringify(this[optionHandle]));
        for(var i = result.length -1; i >= result.length - 1 - this.showMaxChart; i--) {
          newData[0].data.push(result[i].value); 
          newOptions.xaxis.categories.push(result[i].name); 
        }
        this[dataHandle] = newData; 
        this[optionHandle] = newOptions;
      }, 
      parseValues(array) {
        array.forEach(element => {
          element.value = parseInt(element.value, 10); 
        }); 
        return array; 
      }
    }
  }
</script>

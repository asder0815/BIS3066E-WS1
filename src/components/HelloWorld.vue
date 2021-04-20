<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-row>
          <v-col> <v-text-field v-model="blockbusterGross" label="Blockbuster Grossing" outlined :disabled=dataCleaned></v-text-field> </v-col>
          <v-col> <v-text-field v-model="maxBudget" label="Maximum Budget" outlined :disabled=dataCleaned></v-text-field> </v-col>
          <v-col> <v-text-field v-model="maxGrossing" label="Maximum Grossing" outlined :disabled=dataCleaned></v-text-field> </v-col>
          <v-col> <v-text-field v-model="minProfitMargin" label="Minimum Profit Margin" outlined :disabled=dataCleaned></v-text-field> </v-col>
          <v-col> <v-text-field v-model="maxProfitMargin" label="Maximum Profit Margin" outlined :disabled=dataCleaned></v-text-field> </v-col>
        </v-row>
        <v-btn elevation="2" @click="cleanData" :disabled=dataCleaned>Clean Data</v-btn>
      </v-col>  
    </v-row>

    <v-row>
      <v-col> <v-text-field v-model="profitMarginTarget" label="Target Profit Margin" outlined :disabled=dataAnalyzed></v-text-field> </v-col>
      <v-col> <v-text-field v-model="budgetInterval" label="Budget Intervals" outlined :disabled=dataAnalyzed></v-text-field> </v-col>
      <v-col> <v-text-field v-model="showMaxChart" label="Show Top" outlined :disabled=dataAnalyzed></v-text-field> </v-col>
      <v-col> <v-text-field v-model="maxDecimals" label="Max Decimals" outlined :disabled=dataAnalyzed></v-text-field> </v-col>
      <v-col> <v-text-field v-model="minAmountForAverage" label="Minumum amount of data sets for average" outlined :disabled=dataAnalyzed></v-text-field> </v-col>
    </v-row>
    <v-row class="text-center">
      <v-col cols="12">
        <v-btn elevation="2" @click="analyzeData" :disabled='!dataCleaned || dataAnalyzed'>Analyze Data</v-btn>
      </v-col>
    </v-row>

    <!--v-btn elevation="2" @click="analyzeTextAtrribute(['actor_1_name','actor_2_name','actor_3_name'], 'actorValueChart')" :disabled=!dataCleaned>Actor Values</v-btn>
    <v-btn elevation="2" @click="analyzeTextAtrribute(['director_name'], 'directorValueChart')" :disabled=!dataCleaned>Director Values</v-btn>
    <v-btn elevation="2" @click="analyzeKeywords('genres', 'genreValueChart')" :disabled=!dataCleaned>Genres</v-btn>
    <v-btn elevation="2" @click="analyzeKeywords('plot_keywords', 'plotValueChart')" :disabled=!dataCleaned>Plot</v-btn>
    <v-btn elevation="2" @click="analyzeKeywordProfit('genres', 'genreProfitChart')" :disabled=!dataCleaned>Genres Profit</v-btn>
    <v-btn elevation="2" @click="analyzeKeywordProfit('plot_keywords', 'plotProfitChart')" :disabled=!dataCleaned>Plot Profit</v-btn>
    <v-btn elevation="2" @click="analyzeDiminishingReturns('revenueProfitGraph')" :disabled=!dataCleaned>Update Graph</v-btn>
    <v-btn elevation="2" @click="analyzeTextAtrributeAverageProfit(['actor_1_name','actor_2_name','actor_3_name'], 'actorProfitMarginChart')" :disabled=!dataCleaned>Actor Margins</v-btn>
    <v-btn elevation="2" @click="analyzeTextAtrributeAverageProfit(['director_name'], 'directorProfitMarginChart')" :disabled=!dataCleaned>Director Margins</v-btn-->

    <apexchart type="line" height="350" :options="revenueProfitGraphOptions" :series="revenueProfitGraphData"></apexchart>
    <apexchart ref="actorValueChart" type="bar" height="350" :options="actorValueChartOptions" :series="actorValueChartData"></apexchart>
    <apexchart ref="directorValueChart" type="bar" height="350" :options="directorValueChartOptions" :series="directorValueChartData"></apexchart>
    <apexchart ref="genreValueChart" type="bar" height="350" :options="genreValueChartOptions" :series="genreValueChartData"></apexchart>
    <apexchart ref="plotValueChart" type="bar" height="350" :options="plotValueChartOptions" :series="plotValueChartData"></apexchart>
    <apexchart ref="genreProfitChart" type="bar" height="350" :options="genreProfitChartOptions" :series="genreProfitChartData"></apexchart>
    <apexchart ref="plotProfitChart" type="bar" height="350" :options="plotProfitChartOptions" :series="plotProfitChartData"></apexchart>
    <apexchart ref="actorProfitMarginChart" type="bar" height="350" :options="actorProfitMarginChartOptions" :series="actorProfitMarginChartData"></apexchart>
    <apexchart ref="directorProfitMarginChart" type="bar" height="350" :options="directorProfitMarginChartOptions" :series="directorProfitMarginChartData"></apexchart>

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
      dataAnalyzed: false, 
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
      minAmountForAverage: 4, 
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
      revenueProfitGraphData: [{ name: 'Average revenue', data: [] }, { name: 'Average profit', data: [] }],
      revenueProfitGraphOptions: {
        chart: { height: 350, type: 'line', zoom: { enabled: false } },
        dataLabels: { enabled: false },
        stroke: { curve: 'straight' },
        title: { text: 'Diminishing returns of budget', align: 'left' },
        grid: { row: { colors: ['#f3f3f3', 'transparent'], opacity: 0.5 }, },
        xaxis: { categories: [] }
      },
      actorValueChartData: [{ name: 'Weighted actor value', data: [] }],
      actorValueChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Actor Value - What actors are starring in the most successful movies?' },
        xaxis: { categories: [] }
      }, 
      directorValueChartData: [{ name: 'Weighted director value', data: [] }],
      directorValueChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Director Value- Which directors are directing the most successful movies?' },
        xaxis: { categories: [] }
      }, 
      genreValueChartData: [{ name: 'Weighted genre value', data: [] }],
      genreValueChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Genre Value - What genres are the most successful movies?' },
        xaxis: { categories: [] }
      }, 
      plotValueChartData: [{ name: 'Weighted plot value', data: [] }],
      plotValueChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Plot Keyword Value - What plot do the most successful movies have?' },
        xaxis: { categories: [] }
      }, 
      genreProfitChartData: [{ name: 'Chance to surpass profit margin', data: [] }],
      genreProfitChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Genre Profit Chance - What chance to break the target profit margin does each genre have?' },
        xaxis: { categories: [] }
      }, 
      plotProfitChartData: [{ name: 'Chance to surpass profit margin', data: [] }],
      plotProfitChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Plot Keyword Profit Chance - What chance to break the target profit margin does each plot have?' },
        xaxis: { categories: [] }
      },
      actorProfitMarginChartData: [{ name: 'Chance to surpass profit margin', data: [] }],
      actorProfitMarginChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Actor Profit Margin - What average profit margin does a movie with a specific actor have?' },
        xaxis: { categories: [] }
      },
      directorProfitMarginChartData: [{ name: 'Chance to surpass profit margin', data: [] }],
      directorProfitMarginChartOptions: {
        chart: { type: 'bar', height: 350 },
        plotOptions: { bar: { borderRadius: 4 } },
        dataLabels: { enabled: false },
        title: { text: 'Director Profit Margin - What average profit margin does a movie with a specific director have?' },
        xaxis: { categories: [] }
      }
    }),
    methods: {
      cleanData() {
        var cleanStepOne = []; 
        this.movieData.forEach(movie => {
          if(this.movieValid(movie)) cleanStepOne.push(movie);  
        }); 
        var cleanStepTwo = []; 
        var linkSet = new Set(); 
        cleanStepOne.forEach(movie => {
          if(!(movie.movie_imdb_link != "" && linkSet.size === linkSet.add(movie.movie_imdb_link).size)
           && !(movie.movie_title != "" && linkSet.size === linkSet.add(movie.movie_title).size)) cleanStepTwo.push(movie); 
        });
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
                amount: 1,
                value: 0}); 
            }); 
          }
          keywordList.forEach(keyword => {
            if(result.some(e => e.name === keyword.name)) {
              var target = result[result.findIndex(e => e.name === keyword.name)]; 
              target.above += keyword.above;
              target.below += keyword.below; 
              target.amount++; 
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
        result = this.filterAmount(result); 
        this.updateValueGraph(result, false, chartName); 
      },
      analyzeDiminishingReturns(chartName) {
        var result = []; 
        this.cleanedData.forEach(movie => {
          if(result.some(e => e.interval === Math.floor(movie.budget / this.budgetInterval))) {
            var target = result[result.findIndex(e => e.interval === Math.floor(movie.budget / this.budgetInterval))]; 
            target.amount++; 
            target.sum += movie.gross; 
            target.value = target.sum / target.amount; 
            target.sumProfit += movie.gross - movie.budget; 
            target.valueProfit = target.sumProfit / target.amount; 
          }
          else {
            result.push({
              name: Math.floor(movie.budget / this.budgetInterval), 
              interval: Math.floor(movie.budget / this.budgetInterval), 
              amount: 1, 
              sum: movie.gross, 
              value: movie.gross, 
              sumProfit: movie.gross - movie.budget, 
              valueProfit: movie.gross - movie.budget
            }); 
          }
        });  
        this.fixAttributeToDecimals(result, 'value', 2); 
        this.fixAttributeToDecimals(result, 'valueProfit', 2); 
        this.updateValueGraph(result, false, chartName, true);  
        this.updateValueGraph(result, false, chartName, true, 1, true, 'valueProfit');  
      },
      analyzeTextAtrributeAverageProfit(attributeList, chartName) {
        var result = []; 
        this.cleanedData.forEach(movie => {
          var valueList = []; 
          attributeList.forEach(valueNr => {
            if(movie[valueNr] != "") valueList.push({name: movie[valueNr], sum: movie.gross / movie.budget, amount: 1, value: movie.gross / movie.budget}); 
          }); 
          valueList.forEach(value => {
            if(result.some(e => e.name === value.name)) {
              var target = result[result.findIndex(e => e.name === value.name)]; 
              target.sum += value.sum; 
              target.amount++; 
              target.value = target.sum / target.amount; 
            }
            else result.push(value); 
          });
        }); 
        result = this.filterAmount(result); 
        this.updateValueGraph(result, false, chartName); 
      }, 
      sortByValue(a,b) {
        return a.value === b.value ? 0 : a.value > b.value ? 1 : -1; 
      },
      sortByInterval(a,b) {
        return a.interval === b.interval ? 0 : a.interval > b.interval ? -1 : 1; 
      },
      normalizeByValue(array) {
        array.sort(this.sortByValue); 
        var max = array[array.length-1].value; 
        array.forEach(element => {
          element.value = (element.value * 100 / max).toFixed(this.maxDecimals); 
        }); 
        return array; 
      }, 
      updateValueGraph(result, normalize, chartName, attribute = 'value', dataSet = 0, custom = false, customAttribute = '') {
        result = this.parseValues(result, attribute); 
        if(normalize) result = this.normalizeByValue(result); 
        else attribute === 'value' ? result.sort(this.sortByValue) : result.sort(this.sortByInterval); 
        var dataHandle = chartName + "Data"; 
        var optionHandle = chartName + "Options";
        var newData = JSON.parse(JSON.stringify(this[dataHandle]));
        var newOptions = JSON.parse(JSON.stringify(this[optionHandle]));
        for(var i = result.length -1; i >= this.clamp(result.length - 1 - this.showMaxChart, 0); i--) {
          if(!custom) newData[dataSet].data.push(result[i].value); 
          else newData[dataSet].data.push(result[i][customAttribute]);
          newOptions.xaxis.categories.push(result[i].name); 
        }
        this[dataHandle] = newData; 
        this[optionHandle] = newOptions;
      }, 
      parseValues(array, attribute) {
        array.forEach(element => {
          element[attribute] = parseInt(element[attribute], 10); 
        }); 
        return array; 
      }, 
      filterAmount(array) {
        var result = []; 
        array.forEach(element => {
          if(element.amount >= this.minAmountForAverage) result.push(element); 
        });  
        return result; 
      },
      movieValid(movie) {
        return movie.name != "" && movie.budget != "" && movie.gross != "" 
          && movie.gross <= this.maxGrossing && movie.budget <= this.maxBudget
          && movie.gross / movie.budget <= this.maxProfitMargin && movie.gross / movie.budget >= this.minProfitMargin
      }, 
      clamp(value, min, max) {
        return value <= min ? min : value >= max ? max : value;
      }, 
      fixAttributeToDecimals(array, attribute, decimals) {
        array.forEach(element => {
          element[attribute] = element[attribute].toFixed(decimals); 
        });
      }, 
      analyzeData() {
        this.dataAnalyzed = true; 
        this.analyzeTextAtrribute(['actor_1_name','actor_2_name','actor_3_name'], 'actorValueChart'); 
        this.analyzeTextAtrribute(['director_name'], 'directorValueChart'); 
        this.analyzeKeywords('genres', 'genreValueChart'); 
        this.analyzeKeywords('plot_keywords', 'plotValueChart'); 
        this.analyzeKeywordProfit('genres', 'genreProfitChart'); 
        this.analyzeKeywordProfit('plot_keywords', 'plotProfitChart'); 
        this.analyzeDiminishingReturns('revenueProfitGraph'); 
        this.analyzeTextAtrributeAverageProfit(['actor_1_name','actor_2_name','actor_3_name'], 'actorProfitMarginChart'); 
        this.analyzeTextAtrributeAverageProfit(['director_name'], 'directorProfitMarginChart'); 
      }
    }
  }
</script>

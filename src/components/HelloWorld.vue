<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-btn elevation="2" @click="cleanData" :disabled=dataCleaned>Clean Data</v-btn>
        <v-checkbox v-model="fullReport" :label="`Full report: ${fullReport.toString()}`" :disabled=dataCleaned></v-checkbox>
        <v-checkbox v-model="onlyBlockbusters" :label="`Only evaluate blockbusters: ${onlyBlockbusters.toString()}`" :disabled=dataCleaned></v-checkbox>
      </v-col>  
    </v-row>
    <v-btn elevation="2" @click="analyzeActors" :disabled=!dataCleaned>Actors</v-btn>
    <v-btn elevation="2" @click="analyzeTextAtrribute(['director_name'])" :disabled=!dataCleaned>Directors</v-btn>
    <v-btn elevation="2" @click="analyzeKeywords('genres')" :disabled=!dataCleaned>Genres</v-btn>
    <v-btn elevation="2" @click="analyzeKeywords('plot_keywords')" :disabled=!dataCleaned>Plot</v-btn>
    <v-btn elevation="2" @click="analyzeKeywordProfit('genres')" :disabled=!dataCleaned>Genres Profit</v-btn>
    <v-btn elevation="2" @click="analyzeKeywordProfit('plot_keywords')" :disabled=!dataCleaned>Plot Profit</v-btn>
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

  </v-container>
</template>

<script>
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
      maxProfitMargin: 25, 
      minProfitMargin: 0.01,
      profitMarginTarget: 2, 
      budgetInterval: 25000000,
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
      analyzeTextAtrribute(attributeList) {
        var result = []; 
        var baseGross = this.onlyBlockbusters ? this.blockbusterGross : 1; 
        this.cleanedData.forEach(movie => {
          var valueList = []; 
          attributeList.forEach(valueNr => {
            if(movie[valueNr] != "") valueList.push({name: movie[valueNr], value: movie.gross / baseGross}); 
          }); 
          valueList.forEach(value => {
            if(result.some(e => e.name === value.name)) {
              result[result.findIndex(e => e.name === value.name)].value += value.value; 
            }
            else {
              result.push(value); 
            }
          });
        }); 
        result.sort(this.sortByValue); 
        result.forEach(value => {
          console.log(value.name + " Value: " + value.value); 
        }); 
      }, 
      analyzeActors() {
        var result = []; 
        var baseGross = this.onlyBlockbusters ? this.blockbusterGross : 1; 
        this.cleanedData.forEach(movie => {
          var actorList = []; 
          ["actor_1_name", "actor_2_name", "actor_3_name"].forEach(actorNr => {
            if(movie[actorNr] != "") actorList.push({name: movie[actorNr], value: movie.gross / baseGross}); 
          }); 
          actorList.forEach(actor => {
            if(result.some(e => e.name === actor.name)) {
              result[result.findIndex(e => e.name === actor.name)].value += actor.value; 
            }
            else {
              result.push(actor); 
            }
          });
        }); 
        result.sort(this.sortByValue); 
        result.forEach(actor => {
          console.log(actor.name + " Value: " + actor.value); 
        }); 
      }, 
      analyzeKeywords(attribute) {
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
            if(result.some(e => e.name === keyword.name)) {
              result[result.findIndex(e => e.name === keyword.name)].value += keyword.value; 
            }
            else {
              result.push(keyword); 
            }
          });
        }); 
        result.sort(this.sortByValue); 
        result.forEach(keyword => {
          console.log(keyword.name + " Value: " + keyword.value); 
        }); 
      }, 
      analyzeKeywordProfit(attribute) {
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
        result.sort(this.sortByValue); 
        result.forEach(keyword => {
          console.log(keyword.name + " has a " + keyword.value * 100 + "% chance to exceed a profit margin of " + this.profitMarginTarget + "with " + keyword.above + " values above and " + keyword.below + " values below"); 
        }); 
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
    }
  }
</script>

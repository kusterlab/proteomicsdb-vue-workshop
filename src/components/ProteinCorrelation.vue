<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-text-field 
            v-model="geneName1"
            label="Gene name 1"
            placeholder="EGFR"
        ></v-text-field>
        <v-text-field 
            v-model="geneName2"
            label="Gene name 2"
            placeholder="EGFLAM"
        ></v-text-field>
        <v-btn
            @click="updateProteinExpressions">
            Get Protein Expressions
        </v-btn>
      </v-col>
    </v-row>
    <v-row class="text-center">
      <v-col cols="12">
        <div id="protein_scatterplot"></div>
      </v-col>
    </v-row>
    <v-row class="text-center">
      <v-col cols="6">
        <span>Protein ID: {{ proteinId1 }}</span>
        <v-simple-table>
            <template v-slot:default>
              <thead>
                <tr>
                  <th class="text-left">
                    Sample ID
                  </th>
                  <th class="text-left">
                    Expression
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="item in proteinExpressions1"
                  :key="item.SAMPLE_ID"
                >
                  <td>{{ item.SAMPLE_ID }}</td>
                  <td>{{ item.EXPRESSION }}</td>
                </tr>
              </tbody>
            </template>
        </v-simple-table>
      </v-col>
      <v-col cols="6">
        <span>Protein ID: {{ proteinId2 }}</span>
        <v-simple-table>
            <template v-slot:default>
              <thead>
                <tr>
                  <th class="text-left">
                    Sample ID
                  </th>
                  <th class="text-left">
                    Expression
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="item in proteinExpressions2"
                  :key="item.SAMPLE_ID"
                >
                  <td>{{ item.SAMPLE_ID }}</td>
                  <td>{{ item.EXPRESSION }}</td>
                </tr>
              </tbody>
            </template>
        </v-simple-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'
import * as d3 from "d3v4";

export default {
    name: 'ProteinCorrelation',

    data: () => ({
      geneName1: "",
      geneName2: "",
      proteinId1: "",
      proteinId2: "",
      proteinExpressions1: [],
      proteinExpressions2: []
    }),
    
    methods: {
        updateProteinExpressions: function() {
            const promise1 = this.updateProteinId(this.geneName1).then(response => {
                this.proteinId1 = response
                return this.getProteinExpressions(this.proteinId1)
            })
            const promise2 = this.updateProteinId(this.geneName2).then(response => {
                this.proteinId2 = response
                return this.getProteinExpressions(this.proteinId2)
            })
            Promise.all([promise1, promise2]).then((response) => {
                this.proteinExpressions1 = response[0]
                this.proteinExpressions2 = response[1]
                this.plotProteinExpression()
            })
        },
        updateProteinId: function(geneName) {
            let proteinId = -1
            return axios
                .get("https://www.proteomicsdb.org/proteomicsdb/logic/api_v2/api.xsodata/Protein", 
                    { params : { '$filter' : `GENE_NAME eq '${geneName}' and DATABASE eq 'sp' and PARENT_PROTEIN_ID eq PROTEIN_ID and DECOY eq 0`,
                                 '$select' : 'PROTEIN_ID', 
                                 '$format' : 'json' } 
                    }
                )
                .then(response => {
                  if (response.data.d.results.length > 0) {  
                    proteinId = response.data.d.results[0].PROTEIN_ID
                  }
                  return proteinId
                })
                .catch(error => {
                  console.log(error)
                })
            
        },
        getProteinExpressions: function(proteinId) {
            let proteinExpressions = []
            return axios
                .get(`https://www.proteomicsdb.org/proteomicsdb/logic/api_v2/api.xsodata/Protein(${proteinId})/ProteinExpression`,
                    { params : { '$filter' : 'CALCULATION_METHOD eq 0',
                                 '$select' : 'SAMPLE_ID,EXPRESSION',
                                 '$format' : 'json' }
                    }
                )
                .then(response => {
                  if (response.data.d.results.length > 0) {  
                    proteinExpressions = response.data.d.results
                  }
                  return proteinExpressions
                })
                .catch(error => {
                  console.log(error)
                })
        },
        getExpressionInCommonSamples: function() {
            let proteinExpressionsBySampleId1 = Object.assign({}, ...this.proteinExpressions1.map((x) => ({[x.SAMPLE_ID]: x.EXPRESSION})));
            let proteinExpressionsBySampleId2 = Object.assign({}, ...this.proteinExpressions2.map((x) => ({[x.SAMPLE_ID]: x.EXPRESSION})));
            let expressionsInCommonSamples = []
            for (const [key, value] of Object.entries(proteinExpressionsBySampleId1)) {
                if (key in proteinExpressionsBySampleId2) {
                    expressionsInCommonSamples.push({ sampleId : key, proteinExpression1: value, proteinExpression2: proteinExpressionsBySampleId2[key]});
                }
            }
            return expressionsInCommonSamples
        },
        plotProteinExpression: function() {
            let data = this.getExpressionInCommonSamples()
            console.log(data)
            // set the dimensions and margins of the graph
            var margin = {top: 10, right: 30, bottom: 30, left: 60},
                width = 460 - margin.left - margin.right,
                height = 400 - margin.top - margin.bottom;
            
            d3.select("#protein_scatterplot").select("svg").remove()
            
            // append the svg object to the body of the page
            var svg = d3.select("#protein_scatterplot")
              .append("svg")
                .attr("width", width + margin.left + margin.right)
                .attr("height", height + margin.top + margin.bottom)
              .append("g")
                .attr("transform",
                      "translate(" + margin.left + "," + margin.top + ")");

            // Add X axis
            var x = d3.scaleLinear()
              .domain([0, 10])
              .range([ 0, width ]);
            svg.append("g")
              .attr("transform", "translate(0," + height + ")")
              .call(d3.axisBottom(x));

            // Add Y axis
            var y = d3.scaleLinear()
              .domain([0, 10])
              .range([ height, 0]);
            svg.append("g")
              .call(d3.axisLeft(y));

            // Add dots
            svg.append('g')
              .selectAll("dot")
              .data(data)
              .enter()
              .append("circle")
                .attr("cx", function (d) { return x(d.proteinExpression1); } )
                .attr("cy", function (d) { return y(d.proteinExpression2); } )
                .attr("r", 1.5)
                .style("fill", "#69b3a2")

        }
    },

    mounted() {
        
    }
}
</script>

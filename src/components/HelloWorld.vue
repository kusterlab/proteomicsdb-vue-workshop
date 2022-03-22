<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="4">
        <v-text-field 
            v-model="geneName1"
            label="Gene name 1"
            placeholder="EGFR"
            @change="updateProteinId"
         ></v-text-field>
         <span>Protein id 1: {{ proteinId1 }}</span>
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
       <v-col cols="4">
         <v-text-field 
            v-model="geneName2"
            label="Gene name 2"
            placeholder="EGFLAM"
            @change="updateProteinId"
         ></v-text-field>
         <span>Protein id 2: {{ proteinId2 }}</span>
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
        <v-col cols="4">
         <span>Overlap</span>
         <v-simple-table>
            <template v-slot:default>
              <thead>
                <tr>
                  <th class="text-left">
                    Sample ID
                  </th>
                  <th class="text-left">
                    Expression 1
                  </th>
                  <th class="text-left">
                    Expression 2
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="item in proteinExpressionsCommon"
                  :key="item.sampleId"
                >
                  <td>{{ item.sampleId }}</td>
                  <td>{{ item.proteinExpression1 }}</td>
                  <td>{{ item.proteinExpression2 }}</td>
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

export default {
    name: 'ProteinCorrelation',
    data: () => ({
      geneName1: "",
      geneName2: "",
      proteinId1: "",
      proteinId2: "",
      proteinExpressions1: [],
      proteinExpressions2: [],
      proteinExpressionsCommon: []
    }),
    methods: {
        updateProteinId() {
            let promise1 = this.getProteinId(this.geneName1).then(response => {
                this.proteinId1 = response
                return this.getProteinExpressions(this.proteinId1)
            })
            let promise2 = this.getProteinId(this.geneName2).then(response => {
                this.proteinId2 = response
                return this.getProteinExpressions(this.proteinId2)
            })
            Promise.all([promise1, promise2]).then((response) => {
                this.proteinExpressions1 = response[0]
                this.proteinExpressions2 = response[1]
                this.proteinExpressionsCommon = this.getExpressionInCommonSamples()
            })
        },
        getProteinId(geneName) {
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
        getProteinExpressions(proteinId) {
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
    },
}
</script>

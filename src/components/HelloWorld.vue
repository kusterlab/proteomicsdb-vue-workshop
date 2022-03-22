<template>
  <v-container>
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
  </v-container>
</template>

<script>
import axios from 'axios'

export default {
    name: 'ProteinCorrelation',
    data: () => ({
      geneName1: "",
      proteinId1: "",
      proteinExpressions1: []
    }),
    methods: {
        updateProteinId() {
            let promise1 = this.getProteinId(this.geneName1).then(response => {
                this.proteinId1 = response
                return this.getProteinExpressions(this.proteinId1)
            })
            promise1.then(response => {
                this.proteinExpressions1 = response
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
        }
    },
}
</script>

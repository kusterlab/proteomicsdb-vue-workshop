<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-text-field 
            v-model="geneName"
            label="Gene name"
            placeholder="EGFR"
        ></v-text-field>
        <v-btn
            @click="updateProteinId">
            Get Protein ID
        </v-btn>
      </v-col>
    </v-row>
    <v-row class="text-center">
      <v-col cols="12">
        <span>Protein ID: {{ proteinId }}</span>
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
                  v-for="item in proteinExpressions"
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

export default {
    name: 'ProteinCorrelation',

    data: () => ({
      geneName: "",
      proteinId: "",
      proteinExpressions: []
    }),
    
    methods: {
        updateProteinId: function() {
            axios
                .get("https://www.proteomicsdb.org/proteomicsdb/logic/api_v2/api.xsodata/Protein", 
                    { params : { '$filter' : `GENE_NAME eq '${this.geneName}' and DATABASE eq 'sp' and PARENT_PROTEIN_ID eq PROTEIN_ID and DECOY eq 0`,
                                 '$select' : 'PROTEIN_ID', 
                                 '$format' : 'json' } 
                    }
                )
                .then(response => {
                  if (response.data.d.results.length > 0) {  
                    this.proteinId = response.data.d.results[0].PROTEIN_ID
                    this.getProteinExpression()
                  } else {
                    this.proteinId = -1
                  }
                })
                .catch(error => {
                  console.log(error)
                })
        },
        getProteinExpression: function() {
            axios
                .get(`https://www.proteomicsdb.org/proteomicsdb/logic/api_v2/api.xsodata/Protein(${this.proteinId})/ProteinExpression`,
                    { params : { '$filter' : 'CALCULATION_METHOD eq 0',
                                 '$select' : 'SAMPLE_ID,EXPRESSION',
                                 '$format' : 'json' }
                    }
                )
                .then(response => {
                  if (response.data.d.results.length > 0) {  
                    this.proteinExpressions = response.data.d.results
                  } else {
                    this.proteinExpressions = []
                  }
                })
                .catch(error => {
                  console.log(error)
                })
        }
    },

    mounted() {
        
    }
}
</script>

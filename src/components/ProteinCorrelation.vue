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
        {{ proteinId }}
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
      proteinId: ""
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
                  console.log(response.data.d.results)
                  if (response.data.d.results.length > 0) {  
                    this.proteinId = response.data.d.results[0].PROTEIN_ID
                  } else {
                    this.proteinId = -1
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

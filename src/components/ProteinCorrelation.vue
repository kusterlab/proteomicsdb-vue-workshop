<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
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
      proteinId: ""
    }),

    mounted() {
        let geneName = 'EGFR'
        axios
            .get("https://www.proteomicsdb.org/proteomicsdb/logic/api_v2/api.xsodata/Protein", 
                { params : { '$filter' : `GENE_NAME eq '${geneName}' and DATABASE eq 'sp' and PARENT_PROTEIN_ID eq PROTEIN_ID and DECOY eq 0`,
                             '$select' : 'PROTEIN_ID', 
                             '$format' : 'json' } 
                }
            )
            .then(response => {
              if (response.data.d.results.length > 0) {
                this.proteinId = response.data.d.results[0].PROTEIN_ID
              }
            })
            .catch(error => {
              console.log(error)
            })
    }
}
</script>

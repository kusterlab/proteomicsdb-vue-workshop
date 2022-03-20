<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'

export default {
    name: 'ProteinCorrelation',

    data: () => ({
      
    }),

    mounted() {
        axios
            .get("https://www.proteomicsdb.org/proteomicsdb/logic/api_v2/api.xsodata/Protein?$filter=GENE_NAME eq 'EGFR'&$format=json")
            .then(response => {
              console.log(response)
            })
            .catch(error => {
              console.log(error)
            })
    }
}
</script>

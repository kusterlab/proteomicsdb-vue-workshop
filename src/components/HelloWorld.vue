<template>
  <v-container>
    <v-text-field 
        v-model="geneName1"
        label="Gene name 1"
        placeholder="EGFR"
        @change="updateProteinId"
     ></v-text-field>
     <span>{{ proteinId1 }}</span>
  </v-container>
</template>

<script>
import axios from 'axios'

export default {
    name: 'ProteinCorrelation',
    data: () => ({
      geneName1: "",
      proteinId1: ""
    }),
    methods: {
        updateProteinId() {
            this.getProteinId(this.geneName1).then(response => {
                this.proteinId1 = response
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
    }
}
</script>

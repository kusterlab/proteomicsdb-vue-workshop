<template>
  <v-container>
    <v-text-field 
        v-model="geneName1"
        label="Gene name 1"
        placeholder="EGFR"
     ></v-text-field>
     <span>{{ geneName1 }}</span>
  </v-container>
</template>

<script>
export default {
    name: 'ProteinCorrelation',
    data: () => ({
      geneName1: "",
    }),
}
</script>

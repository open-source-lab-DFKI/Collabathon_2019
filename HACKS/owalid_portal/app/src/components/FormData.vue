<template>
  <div>
    <v-container fluid>
      <div v-if="!loading">
        <v-row>
          <v-col cols="12">
            <v-row align="center" justify="center" class="mr-12 ml-12">
              <v-select
                v-model="selectCountry"
                :items="itemsCountry"
                @change="changeCountry"
                label="Country"
                class="ma-3"
              ></v-select>
              <v-select
                v-model="selectSector"
                @change="changeSector"
                :items="itemsSector"
                label="Sector"
                class="ma-3"
              ></v-select>
              <v-select
                v-model="selectGas"
                @change="changeGas"
                :items="itemsGas"
                label="Gas"
                class="ma-3"
              ></v-select>
              <v-select
                v-model="selectUnit"
                @change="changeUnit"
                :items="itemsUnit"
                label="Unit"
                class="ma-3"
              ></v-select>
            </v-row>
          </v-col>
          <!-- <v-range-slider v-model="selectYears"></v-range-slider> -->
        </v-row>
      </div>
      <div v-else>
        <v-progress-circular indeterminate color="primary"></v-progress-circular>
      </div>
    </v-container>
  </div>
</template>

<script>
import { country, sector, gas, unit } from "../utils/fieldForm";
export default {
  name: "formData",
  data() {
    return {
      loading: true,
      dataChart: "",
      countryData: "",
      selectCountry: "Afghanistan",
      selectSector: "",
      selectGas: "",
      selectUnit: "",
      selectYears: "",
      itemsCountry: country,
      itemsSector: sector,
      itemsGas: gas,
      itemsUnit: unit
    };
  },
  async created() {
    try {
      const data = { country: this.selectCountry };
      let res = await axios.post(
        `${process.env.VUE_APP_API_URL}/data/getDataWithCountry`,
        data
      );
      this.loading = false;
      this.dataChart = res.data.data;
      this.countryData = res.data.data;
      this.$emit("updateData", this.dataChart);
    } catch (error) {
      console.error(error);
    }
  },
  methods: {
    beforeChange(changeValue) {
      let tmpArray = this.countryData;
      tmpArray = tmpArray.filter(k => {
        if (changeValue === "Sector") {
          return (
            k.gas == (this.selectGas || k.gas) &&
            k.unit == (this.selectUnit || k.unit)
          );
        } else if (changeValue === "Unit") {
          return (
            k.gas == (this.selectGas || k.gas) &&
            k.sector == (this.selectSector || k.sector)
          );
        } else if (changeValue === "Gas") {
          return (
            k.sector == (this.selectSector || k.sector) &&
            k.unit == (this.selectUnit || k.unit)
          );
        } else if (changeValue === "Country") {
          return (
            k.sector == (this.selectSector || k.sector) &&
            k.unit == (this.selectUnit || k.unit) &&
            k.gas == (this.selectGas || k.gas)
          );
        }
      });
      return tmpArray;
    },
    async changeCountry() {
      const data = { country: this.selectCountry };
      let res = await axios.post(
        `${process.env.VUE_APP_API_URL}/data/getDataWithCountry`,
        data
      );
      this.dataChart = res.data.data;
      this.countryData = this.dataChart;
      this.dataChart = this.beforeChange("Country");
      this.$emit("updateData", this.dataChart);
    },
    async changeSector() {
      const data = { sector: this.selectSector };
      let res = await axios.post(
        `${process.env.VUE_APP_API_URL}/data/getDataWithSector`,
        data
      );
      const tmpData = this.beforeChange("Sector");
      this.dataChart = res.data.data.filter(o =>
        tmpData.find(o2 => o._id === o2._id)
      );
      this.$emit("updateData", this.dataChart);
    },
    async changeGas() {
      const data = { gas: this.selectGas };
      let res = await axios.post(
        `${process.env.VUE_APP_API_URL}/data/getDataWithGas`,
        data
      );
      const tmpData = this.beforeChange("Gas");
      this.dataChart = res.data.data.filter(o =>
        tmpData.find(o2 => o._id === o2._id)
      );
      this.$emit("updateData", this.dataChart);
    },
    async changeUnit() {
      const data = { unit: this.selectUnit };
      let res = await axios.post(
        `${process.env.VUE_APP_API_URL}/data/getDataWithUnit`,
        data
      );
      const tmpData = this.beforeChange("Unit");
      this.dataChart = res.data.data.filter(o =>
        tmpData.find(o2 => o._id === o2._id)
      );
      this.$emit("updateData", this.dataChart);
    }
  }
};
</script>
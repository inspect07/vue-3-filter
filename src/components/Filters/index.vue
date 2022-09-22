<script lang="ts" setup>
import { ref, onMounted, computed, watch } from "@vue/runtime-dom";
import BasicFilter from "./BasicFilter.vue";
import AdvanceFilter from "./AdvanceFilter.vue";
import Countries from "../../assets/countries.json";
import Regions from "../../assets/regions.json";
import { onClickOutside } from "@vueuse/core";

interface defineData {
  name: String;
  selected: Boolean;
  id: any;
}

const showBasic = ref(false);
const showAdvance = ref(false);
const countries = ref([]);
const regions = ref([]);
const basic_filter = ref(null);
const count = ref(0);
const selectedFilter = ref([]);
const searchCountryResult = ref([]);
const searchRegionResult = ref([]);
const allData = ref([]);
const searchResult = ref([]);

const basicFilter = () => {
  showBasic.value = true;
};

const advanceFilter = () => {
  showAdvance.value = true;
};

const closeFilter = () => {
  showAdvance.value = false;
};

const mapJson = () => {
  countries.value = Countries.map((country, index) => ({
    name: country,
    id: index,
    selected: false,
  }));

  regions.value = Regions.map((region, index) => ({
    name: region,
    id: index,
    selected: false,
  }));
};

const limitCountries = computed(() => {
  return countries.value.slice(0, 5);
});

const limitRegions = computed(() => {
  return regions.value.slice(0, 5);
});

const selectSearch = (
  selected: defineData,
  type: "region" | "country" | "all"
) => {
  if (type == "country") {
    let index = countries.value.indexOf(selected);
    let c = countries.value[index];
    c.selected = !c.selected;
    sortSelected("country");
    counter();
  }

  if (type == "region") {
    let index = regions.value.indexOf(selected);
    let r = regions.value[index];
    r.selected = !r.selected;
    sortSelected("region");
    counter();
  }

  if (type == "all") {
    if (!selectedFilter.value.includes(selected)) {
      selectedFilter.value.push(selected);
    }
  }
};

const counter = () => {
  let country = countries.value.filter((c) => {
    return c.selected == true;
  });
  let region = regions.value.filter((c) => {
    return c.selected == true;
  });
  count.value = country.length + region.length;
};

const sortSelected = (type: "region" | "country") => {
  if (type == "country") {
    countries.value = countries.value.sort(
      (a, b) => Number(b.selected) - Number(a.selected)
    );
  } else {
    regions.value = regions.value.sort(
      (a, b) => Number(b.selected) - Number(a.selected)
    );
  }
};

const search = (type: "region" | "country" | "all", text: string) => {
  if (text == "") {
    searchCountryResult.value = [];
    searchRegionResult.value = [];
    searchResult.value = [];
    return;
  }

  let result: any = [];
  if (type == "country") {
    countries.value.map((data) => {
      if (data.name.toLowerCase().includes(text.toLowerCase())) {
        result.push(data);
      }
    });
    searchCountryResult.value = result;
  }

  if (type == "region") {
    regions.value.map((data) => {
      if (data.name.toLowerCase().includes(text.toLowerCase())) {
        result.push(data);
      }
    });
    searchRegionResult.value = result;
  }

  if (type == "all") {
    allData.value.map((data) => {
      if (data.name.toLowerCase().includes(text.toLowerCase())) {
        result.push(data);
      }
    });
    searchResult.value = result;
  }
};

const checkedFilter = (selected: any) => {
  if (!selectedFilter.value.includes(selected)) {
    selectedFilter.value.push(selected);
  } else {
    let index = selectedFilter.value.indexOf(selected);
    selectedFilter.value.splice(index, 1);
  }

  console.log(selectedFilter.value);
};

const actionFilter = (selected: any, type: "remove" | "excluded") => {
  let index = selectedFilter.value.indexOf(selected);

  if (type == "remove") {
    selectedFilter.value[index].exclude = false;
    selectedFilter.value.splice(index, 1);
  }

  if (type == "excluded") {
    selectedFilter.value[index].exclude = true;
  }
};

const setRange = (year: number | string, type: "from" | "to") => {
  if (year == "") return;

  let hasRange = false;
  selectedFilter.value.map((data: any) => {
    hasRange = data.hasOwnProperty("isRange");
  });

  if (!hasRange) {
    selectedFilter.value.push({
      isRange: true,
      year_from: "",
      year_to: "",
    });
  }

  selectedFilter.value.filter((obj: any) => {
    if (obj.isRange) {
      if (type == "from") {
        obj.year_from = year;
      }
      if (type == "to") {
        obj.year_to = year;
      }
    }
  });
};

const clearAll = () => {
  selectedFilter.value = [];
};

onClickOutside(basic_filter, () => {
  showBasic.value = false;
});

onMounted(() => {
  mapJson();
  allData.value = [...countries.value, ...regions.value];
});
</script>

<template>
  <div class="filter-container">
    <div class="filters">
      <div class="--btn-accent" @click="basicFilter">
        <span class="filter-counter" v-if="count > 0">{{ count }}</span>
        <span>Location</span>
      </div>
      <div class="--btn-accent accent-green" @click="advanceFilter">
        <span>Advanced filters</span>
      </div>
    </div>
    <div class="selected-filters"></div>

    <BasicFilter
      :countries="limitCountries"
      :regions="limitRegions"
      :searchCountryResult="searchCountryResult"
      :searchRegionResult="searchRegionResult"
      v-if="showBasic"
      @selectCountryCB="selectSearch"
      @selectRegionCB="selectSearch"
      @searchCB="search"
      ref="basic_filter"
    />
    <AdvanceFilter
      v-if="showAdvance"
      :searchResult="searchResult"
      :selectedFilter="selectedFilter"
      @selectSearch="selectSearch"
      @setRangeCB="setRange"
      @searchCB="search"
      @checkedFilterCB="checkedFilter"
      @actionFilterCB="actionFilter"
      @closeFilterCB="closeFilter"
      @clearAllCB="clearAll"
    />
  </div>
</template>

<style lang="scss" scoped>
.filter-container {
  .filters {
    display: flex;

    .filter-counter {
      display: flex;
      justify-content: center;
      align-items: center;
      position: absolute;
      width: 16px;
      height: 16px;
      background: var(--theme-color);
      border-radius: 25px;
      margin-left: 3.7rem;
      margin-top: -0.7rem;
      font-size: 0.59rem;
      color: var(--primary-white);
    }
  }

  .selected-filters {
    display: flex;
  }
}
</style>

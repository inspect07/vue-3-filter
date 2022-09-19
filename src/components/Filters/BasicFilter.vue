<script lang="ts" setup>
import { defineProps } from "vue";
import { ref, watch, defineEmits } from "@vue/runtime-dom";

const country_key = ref("");
const region_key = ref("");
const emit = defineEmits(["searchCB", "selectCountryCB", "selectRegionCB"]);

const props = defineProps({
  countries: {
    type: Array,
    default: [],
  },
  regions: {
    type: Array,
    default: [],
  },
  searchCountryResult: {
    type: Array,
    default: [],
  },
  searchRegionResult: {
    type: Array,
    default: [],
  },
});

const selectCountry = (country: Object) => {
  emit("selectCountryCB", country, "country");
  country_key.value = "";
};

const selectRegion = (region: Object) => {
  emit("selectRegionCB", region, "region");
  region_key.value = "";
};

watch(
  () => country_key.value,
  () => {
    emit("searchCB", "country", country_key.value);
  }
);

watch(
  () => region_key.value,
  () => {
    emit("searchCB", "region", region_key.value);
  }
);
</script>

<template>
  <div class="basic-filter">
    <div class="menu-title">
      <span>Country</span>
      <span class="icon"><font-awesome-icon icon="filter" /></span>
    </div>
    <div class="search">
      <input type="text" placeholder="Type to Search" v-model="country_key" />
    </div>
    <div class="list-container">
      <ul v-if="props.searchCountryResult.length > 0">
        <li
          v-for="(country, key) in props.searchCountryResult"
          :key="key"
          @click="selectCountry(country)"
        >
          <input type="checkbox" :checked="country.selected" />
          <i class="custom-mark"></i><span>{{ country.name }}</span>
        </li>
      </ul>
      <ul v-else>
        <li
          v-for="(country, key) in props.countries"
          :key="key"
          @click="selectCountry(country)"
        >
          <input type="checkbox" :checked="country.selected" />
          <i class="custom-mark"></i><span>{{ country.name }}</span>
        </li>
      </ul>
    </div>
    <div class="menu-title">
      <span>Region</span>
      <span class="icon"><font-awesome-icon icon="filter" /></span>
    </div>
    <div class="search">
      <input type="text" placeholder="Type to Search" v-model="region_key" />
    </div>
    <div class="list-container">
      <ul v-if="props.searchRegionResult.length > 0">
        <li
          v-for="(region, key) in props.searchRegionResult"
          :key="key"
          @click="selectRegion(region)"
        >
          <input type="checkbox" :checked="region.selected" />
          <i class="custom-mark"></i><span>{{ region.name }}</span>
        </li>
      </ul>
      <ul v-else>
        <li
          v-for="(region, key) in props.regions"
          :key="key"
          @click="selectRegion(region)"
        >
          <input type="checkbox" :checked="region.selected" />
          <i class="custom-mark"></i><span>{{ region.name }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.basic-filter {
  width: 15.5rem;
  position: absolute;
  background: var(--white);
  border: 1px solid var(--primary-light-gray);
  border-radius: 1rem;
  margin-top: 0.5rem;
  display: flex;
  flex-direction: column;

  .menu-title {
    display: flex;
    justify-content: space-between;
    font-weight: 600;
    font-size: 0.9375rem;

    padding: 0.3rem 0.75rem;

    .icon {
      color: var(--theme-color);
    }
  }

  .search {
    display: flex;
    padding: 0.3rem 0;
    padding: 0.3rem 0.75rem;
    padding-bottom: 0.625rem;

    input {
      width: 100%;
      height: 2rem;
      border: 1px solid var(--primary-light-gray);
      border-radius: 2rem;
      padding-left: 1rem;
      outline: none;

      &::placeholder {
        color: var(--primary-light-gray);
      }
    }
  }

  .list-container {
    max-height: 12rem;
    min-height: 12rem;
    overflow: auto;

    ul {
      list-style: none;
      padding-left: 0px;
      margin: 0px;

      li {
        display: flex;
        font-size: 0.875rem;
        font-weight: 300;
        padding: 0.3rem 0.75rem;
        cursor: pointer;

        input {
          width: 1.125rem;
          height: 1.125rem;
          accent-color: #6fbc7d;

          &:checked ~ :after {
            display: block;
          }
        }

        i:after {
          content: "";
          position: absolute;
          display: none;
          left: 21px;
          width: 4px;
          height: 10px;
          border: solid #fff;
          border-width: 0 4px 4px 0;
          -webkit-transform: rotate(40deg);
          -ms-transform: rotate(40deg);
          transform: rotate(40deg);
          margin-top: 3px;
        }

        span {
          padding-left: 0.3rem;
        }

        &:hover {
          background: var(--primary-light-green);
        }
      }
    }
  }
}
</style>

<script lang="ts" setup>
import { ref, onMounted, watch, defineEmits } from "@vue/runtime-dom";
import { defineProps } from "vue";

const sizes = ref([]);
const stages = ref([]);
const search_key = ref("");
const from = ref("");
const to = ref("");

const emit = defineEmits([
  "searchCB",
  "selectSearch",
  "checkedFilterCB",
  "actionFilterCB",
  "setRangeCB",
  "closeFilterCB",
  "clearAllCB",
]);

const props = defineProps({
  searchResult: {
    type: Array,
    default: [],
  },
  selectedFilter: {
    type: Array,
    default: [],
  },
});

const selectResult = (selected: any) => {
  emit("selectSearch", selected, "all");
  search_key.value = "";
};

const checkedFilter = (selected: any) => {
  selected.selected = !selected.selected;
  emit("checkedFilterCB", selected);
};

const excludeSelected = (selected: any) => {
  emit("actionFilterCB", selected, "excluded");
};

const removeSelected = (selected: any) => {
  selected.selected = !selected.selected;
  emit("actionFilterCB", selected, "remove");

  if (selected.isRange) {
    from.value = "";
    to.value = "";
  }
};

const setRange = (year: Number, type: "from" | "to") => {
  emit("setRangeCB", year, type);
};

const clearAll = () => {
  sizes.value.map((obj: any) => {
    obj.selected = false;
  });

  stages.value.map((obj: any) => {
    obj.selected = false;
  });
  from.value = "";
  to.value = "";
  emit("clearAllCB");
};

onMounted(() => {
  sizes.value = [
    { from: 1, to: 10, selected: false },
    { from: 11, to: 50, selected: false },
    { from: 51, to: 100, selected: false },
    { from: 101, to: 200, selected: false },
    { from: 201, to: 500, selected: false },
    { from: 501, to: 1000, selected: false },
    { from: 1001, to: 5000, selected: false },
    { from: 5001, to: 10000, selected: false },
    { from: 10001, to: "", selected: false },
  ];

  stages.value = [
    { stage: "Ideation", selected: false },
    { stage: "Early", selected: false },
    { stage: "Growth", selected: false },
    { stage: "Scaling", selected: false },
  ];
});

watch(
  () => search_key.value,
  () => {
    emit("searchCB", "all", search_key.value);
  }
);

watch(
  () => from.value,
  () => {
    setRange(from.value, "from");
  }
);

watch(
  () => to.value,
  () => {
    setRange(to.value, "to");
  }
);
</script>
<template>
  <div class="advance-filter">
    <div class="sidebar">
      <div class="menu active">OVERVIEW</div>
    </div>
    <div class="filters-container">
      <div class="filters">
        <div class="title">Location</div>
        <div class="search">
          <input
            type="text"
            :class="`${props.searchResult.length > 0 ? 'hasResult' : ''}`"
            placeholder="Type to search"
            v-model="search_key"
          />
          <div class="result" v-if="props.searchResult.length > 0">
            <ul>
              <li
                v-for="(result, key) in props.searchResult"
                :key="key"
                @click="selectResult(result)"
              >
                {{ result.name }}
              </li>
            </ul>
          </div>
        </div>
        <div class="title mt-30">Founded Year</div>
        <div class="date-range">
          <input type="number" placeholder="2019" v-model="from" max="4" />
          <span>To</span>
          <input type="number" placeholder="2022" v-model="to" maxlength="4" />
        </div>
        <div class="title mt-30">Size</div>
        <div class="sizes">
          <ul>
            <li
              v-for="(size, key) in sizes"
              :key="key"
              @click="checkedFilter(size)"
            >
              <input
                type="checkbox"
                :checked="size.selected ? true : false"
              /><i class="custom-mark"></i>
              <span>{{
                size.to == "" ? `${size.from}+` : `${size.from} - ${size.to}`
              }}</span>
            </li>
          </ul>
        </div>
        <div class="title mt-30">Stage</div>
        <div class="stages">
          <ul>
            <li
              v-for="(stage, key) in stages"
              :key="key"
              @click="checkedFilter(stage)"
            >
              <input
                type="checkbox"
                :checked="stage.selected ? true : false"
              /><i class="custom-mark"></i>
              <span>{{ stage.stage }}</span>
            </li>
          </ul>
        </div>
      </div>
      <div class="selected-filter">
        <ul>
          <li
            v-for="(filter, key) in props.selectedFilter"
            :key="key"
            :class="`${filter.exclude ? 'excluded' : ''}`"
          >
            <span v-if="filter.name">&nbsp;{{ filter.name }}&nbsp;</span>
            <span v-if="filter.from !== '' && filter.to == ''"
              >&nbsp;From {{ filter.from }}&nbsp;</span
            >
            <span v-if="filter.to !== '' && filter.from == ''"
              >&nbsp;to {{ filter.to }}&nbsp;</span
            >
            <span v-if="filter.to !== '' && filter.from !== ''"
              >&nbsp;{{ filter.from }} to {{ filter.to }}&nbsp;</span
            >
            <span v-if="filter.stage">&nbsp;{{ filter.stage }}&nbsp;</span>
            <div class="action remove" @click="removeSelected(filter)">
              &#x2715;
            </div>
            <div class="action exclude" @click="excludeSelected(filter)">
              &#8211;
            </div>
          </li>
        </ul>
      </div>
      <div class="footer">
        <div class="footer-buttons">
          <div class="apply --btn-accent">
            <span>Apply</span>
          </div>
          <div class="clear-all --btn-accent" @click="clearAll">
            <span>Clear all</span>
          </div>
          <div class="cancel --btn-accent" @click="emit('closeFilterCB')">
            <span>cancel</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.advance-filter {
  height: 100vh;
  width: 100vw;
  background: #fff;
  position: absolute;
  left: 0;
  top: 0;
  display: flex;
  animation: fadeIn 0.3s;

  .sidebar {
    background: #f8fcfc;
    display: flex;
    flex-direction: column;
    width: 100%;
    max-width: 13%;
    padding: 1.5625rem;

    .menu {
      padding: 0.9375rem 0.3125rem;
    }

    .active {
      color: var(--theme-color);
      font-weight: 600;
    }

    border-right: solid 1px var(--primary-light-gray);
  }

  .filters-container {
    display: flex;
    flex-direction: column;
    width: 100%;

    .filters {
      padding: 1.875rem;
      margin: 0.625rem;
      .title {
        display: flex;
        text-transform: uppercase;
        align-items: center;
        font-size: 0.875rem;
        font-weight: 300;
        margin-bottom: 0.625rem;

        &::after {
          border-bottom: 1px solid hsla(0, 0%, 44%, 0.12);
          content: " ";
          flex: 1 1;
          margin-left: 0.5rem;
        }
      }
      .search {
        max-width: 33.3333333333%;
        display: flex;
        flex-direction: column;
        position: relative;

        .result {
          position: absolute;
          min-height: 36.7812px;
          max-height: 183.906px;
          overflow: auto;
          width: 99.5%;
          display: block;
          margin-top: 30px;
          background: #fff;
          border: 1px solid hsla(0, 0%, 44%, 0.12);
          border-radius: 0 0 8px 8px !important;
          border-top: 0 solid hsla(0, 0%, 44%, 0.12);

          ul {
            padding: 0;
            li {
              display: flex;
              font-size: 0.875rem;
              font-weight: 300;
              padding: 0.4rem 1rem;
              cursor: pointer;

              &:hover {
                background: var(--primary-light-green);
              }
            }
          }
        }

        input {
          // width: 100%;
          height: 2rem;
          border-radius: 2rem;
          padding-left: 1rem;
          outline: none;
          border: 1px solid hsla(0, 0%, 44%, 0.12);

          &.hasResult {
            border-radius: 1rem 1rem 0rem 0rem !important;
          }

          ::placeholder {
            color: hsla(0, 0%, 44%, 0.12);
          }
        }
      }
    }

    .date-range {
      display: flex;
      align-items: center;

      input[type="number"] {
        height: 2rem;
        border-radius: 2rem;
        padding-left: 1rem;
        outline: none;
        border: 1px solid hsla(0, 0%, 44%, 0.12);
        margin: 0px 14px;
        width: 100px;
        appearance: textfield;

        ::placeholder {
          color: hsla(0, 0%, 44%, 0.12);
        }

        &::-webkit-outer-spin-button,
        &::-webkit-inner-spin-button {
          -webkit-appearance: none;
          margin: 0;
        }
      }
    }

    .sizes {
      ul {
        columns: 3;
        list-style: none;
        padding: 0px;

        li {
          cursor: pointer;
          display: flex;
          font-weight: 300;
          color: var(--primary-gray);
          margin-bottom: 0.5rem;

          span {
            padding-left: 10px;
          }

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
            margin-left: -16px;
            width: 4px;
            height: 10px;
            border: solid #fff;
            border-width: 0 4px 4px 0;
            -webkit-transform: rotate(40deg);
            -ms-transform: rotate(40deg);
            transform: rotate(40deg);
            margin-top: 3px;
          }
        }
      }
    }

    .stages {
      ul {
        columns: 4;
        list-style: none;
        padding: 0px;

        li {
          display: flex;
          cursor: pointer;
          font-weight: 300;
          color: var(--primary-gray);
          margin-bottom: 0.5rem;

          span {
            padding-left: 10px;
          }

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
            margin-left: -16px;
            width: 4px;
            height: 10px;
            border: solid #fff;
            border-width: 0 4px 4px 0;
            -webkit-transform: rotate(40deg);
            -ms-transform: rotate(40deg);
            transform: rotate(40deg);
            margin-top: 3px;
          }
        }
      }
    }

    .selected-filter {
      border-top: solid 1px var(--primary-light-gray);
      margin-top: auto;
      min-height: 8vh;

      ul {
        // display: flex;
        list-style: none;
        li {
          display: inline-block;
          cursor: pointer;
          width: auto;
          border: 1px solid var(--primary-light-gray);
          border-radius: 0.25rem;
          padding: 0.3125rem;
          font-weight: 400;
          line-height: 18px;
          margin-right: 0.6rem;
          margin-top: 0.3125rem;
          margin-bottom: 0.3125rem;
          position: relative;

          span {
            padding: 0 0.7rem;
          }

          .action {
            display: none;
            justify-content: center;
            align-items: center;
            position: absolute;
            font-size: 11px;
            background: var(--theme-color);
            width: 17px;
            height: 17px;
            text-align: center;
            border-radius: 50px;
            color: #fff;
            margin-top: 15px;
            margin-left: -3px;
            z-index: 1;
            &.remove {
              margin-top: -1rem;
            }
          }

          &.excluded {
            text-decoration-line: line-through;
          }

          &:hover {
            .action {
              display: inline-block;
            }
          }
        }
      }
    }

    .footer {
      min-height: 8vh;
      border-top: solid 1px var(--primary-light-gray);
      display: flex;
      justify-content: flex-end;

      .footer-buttons {
        display: flex;
        align-items: center;
        justify-content: flex-end;
        text-transform: uppercase;

        .apply {
          background: var(--theme-color);
          border: none;
          color: #fff;
          padding: 0.4rem 1.5rem;
        }

        .clear-all {
          border: 1px solid var(--primary-light-gray);
          padding: 0.4rem 1.5rem;
          color: var(--theme-color);
        }

        .cancel {
          border: none;
          color: var(--theme-color);
          padding: 0.4rem 0.9rem;
        }
      }
    }
  }
}

@keyframes fadeIn {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
</style>

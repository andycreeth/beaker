<template>
  <div class="list--input">
    <div @click="toggleDropdown()" class="list--input-head" :class="this.value.length == 0 && 'list-input-head-empty'">
      <input
        @click="toggleDropdown()"
        :value="this.value"
        id='search'
        :placeholder="this.placeholder"
        @change="changeValue"
      />
    </div>
    <div v-if="showDropdown" class="list--input-dropdown">
      <div
        v-for="option in optionsState"
        class="list--input-dropdown--item"
        :id="option.id"
        v-if="showDropdown"
        @click="selectOption(option)"
      >
        <div class="list--input-dropdown--icon" v-if="icon"> {{ icon }} </div>
        <div class="list--input-dropdown--item_image" v-if="image"> {{ image }} </div>
        <div class="list--input-dropdown--item_label" v-if="typeof option === 'object' && option.label !== undefined"> {{option.label}} </div>
        <div class="list--input-dropdown--item_label" v-if="typeof option === 'object' && option.value !== undefined"> {{option.value}} </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from "vue-property-decorator";

interface Option {
  value: string | number | boolean;
  label: string;
  icon?: string;
  status?: 'enabled' | 'disabled';
  image?: string;
}

@Component({})
export default class FormGroup extends Vue {
  @Prop({default: null})
  options!: Option[];

  @Prop()
  value!: string | number;

  @Prop()
  placeholder!: string;

  showDropdown!: boolean;
  query = null;
  optionsState = [];

  changedQuery: 'yes' | 'no';

  @Prop({default: function() {}})
  beforeOpenDropdownHandler!: Function;

  @Prop()
  onSelectOption!: Function;

  get query() {
    return this.$store.getters.listInputQuery;
  }

  @Watch(this.query)
  mutateQuery() {
    this.$store.state.listInputQueryString = this.query;
  }

  created() {
    if (typeof this.beforeOpenDropdownHandler === 'function') this.beforeOpenDropdownHandler.bind(this);
    if (!this.placeholder) this.placeholder = 'some placeholder';
    this.query = this.value;
    this.occasionally_filter_options_by_query();
    this.optionsState = [...this.options];

    const that = this;
    setInterval(function() {
      that.occasionally_filter_options_by_query();
    }, 10);
  }

  changeValue() {
    var query = $('#search').val();
    this.query = query.toString();
    setTimeout(function() {
      this.changedQuery = 'yes';
    },100);
  }

  occasionally_filter_options_by_query() {
    if (this.changedQuery === 'yes') {
      const newOptionsState = this.options.forEach(function (option) {
        if (!option) option = {};

        const stringToQuery = '';
        if ((option.value + option.label).indexOf(this.query) > -1) {
          return Object.assign({}, option)
        }
      });

      newOptionsState.sort(function (a,b) {
        const aValue = a.value || a.label;
        const bValue = a.value || a.label;
        if (aValue <= bValue) return -1;
        else if (bValue >= aValue) return 1;
        else if (aValue == bValue) return 0;
      })

      this.changedQuery = 'no';

      if (newOptionsState.length == 0) {
        const noMatchOption = {
          label: 'nothing match',
          value: null
        }
        this.selectOption(<Option> noMatchOption);
      }

      return this.optionsState = newOptionsState;
    }
  }

  selectOption(option: Option) {
    $('#search').val(option.value);
    this.$store.state.listInputQueryString = option.value;
    this.toggleDropdown();
    this.onSelectOption(option);
  }

  toggleDropdown(currentlyShowing = false) {
    if (currentlyShowing) this.showDropdown = false;
    if (!currentlyShowing) this.showDropdown = true;
  }

}
</script>

<style lang="less" scoped>
@import "./../styles/Imports";
.list--input {
  padding: 10px;

  .list--input-head {
    margin-top: -10px;
    margin-left: -10px;
    margin-right: -10px;
    width: calc(100% + (2 * 10px));
    border: 2px gray;

    &.list-input-head-empty {

      input {
        border: red;
        border-width: 1;

        &:after {
          background-image: url('some/link/to/warning.icon');
          background: red;
          width: 20px;
          height: 20px;
          position: 'absolute';
        }
      }
    }
  }

  .list--input-dropdown {
    & > * {
      flex: 1;
      line-height: 20px;
      z-index: 2;
      cursor: pointer;
      display: inline;
      margin-bottom: 5px;

      &:Hover {
        margin-left: 10rm;
        color: teal;
        & > * {
          color: teal;
        }
      }
    }
  }
  .list--input-dropdown--item {
    display: flex;
    flex-direction: row-start;
    justify-item: space-between-children;

    .list--input-dropdown--item_label {
      color: white !important;
      min-height: 20px;
    }
  }

}
</style>

<script>
import MultiSelect from './MultiSelect.vue';
import { Container, Draggable } from 'vue-smooth-dnd';
import { commonMixin } from './mixins';

export default {
  name: 'CustomMultiListSelect',
  mixins: [commonMixin],
  render(createElement) {
    return createElement(MultiSelect, {
      props: {
        id: this.id,
        name: this.name,
        options: this.options,
        selectedOptions: this.items,
        isError: this.isError,
        isDisabled: this.isDisabled,
        placeholder: this.placeholder,
        filterPredicate: this.filterPredicate,
      },
      on: {
        select: this.onSelect,
        searchchange: (searchText) => this.$emit('searchchange', searchText),
      },
    });
  },
  props: {
    list: {
      type: Array,
    },
    optionValue: {
      type: String,
    },
    optionText: {
      type: String,
    },
    customText: {
      type: Function,
    },
    selectedItems: {
      type: Array,
    },
  },
  computed: {
    options() {
      return this.list.map((e) => {
        return { value: e[this.optionValue], text: this.buildText(e) };
      });
    },
    items() {
      return this.selectedItems.map((e) => {
        return { value: e[this.optionValue], text: this.buildText(e) };
      });
    },
  },
  methods: {
    buildText(e) {
      if (e[this.optionValue] !== undefined) {
        if (this.customText) {
          return this.customText(e);
        } else {
          return e[this.optionText];
        }
      } else {
        return '';
      }
    },
    onSelect(options, option) {
      if (Object.keys(option).length === 0 && option.constructor === Object) {
        this.$emit('select', options, option);
      } else {
        const items = this.list.filter((e) => {
          return options.find((o) => e[this.optionValue] === o.value);
        });
        const item = this.list.find(
          (e) => e[this.optionValue] === option.value
        );
        this.$emit('select', items, item);
      }
    },
    onDrop({ removedIndex, addedIndex }) {
      const movedItem = this.selectedItems.splice(removedIndex, 1)[0];
      this.selectedItems.splice(addedIndex, 0, movedItem);
      this.$emit('update:selectedItems', this.selectedItems);
    },
  },
  components: {
    MultiSelect,
    Container,
    Draggable,
  },
};
</script>

<template>
  <div>
    <!-- Draggable Container for Selected Items -->
    <container @drop="onDrop">
      <draggable
        v-for="(item, index) in items"
        :key="item.value"
        class="draggable-item"
      >
        <div>
          {{ item.text }}
        </div>
      </draggable>
    </container>

    <!-- Render MultiSelect -->
    <multi-select
      :id="id"
      :name="name"
      :options="options"
      :selected-options="items"
      :is-error="isError"
      :is-disabled="isDisabled"
      :placeholder="placeholder"
      :filter-predicate="filterPredicate"
      @select="onSelect"
      @searchchange="$emit('searchchange', $event)"
    />
  </div>
</template>

<style>
.draggable-item {
  padding: 8px;
  margin: 4px 0;
  background: #f9f9f9;
  border: 1px solid #ccc;
  border-radius: 4px;
  cursor: grab;
}
</style>

<script>
/**
 * @fileoverview A customizable select/dropdown input field component for forms.
 *
 * @description This component is a wrapper around PrimeVue's Select component,
 * providing a standardized way to create select inputs. It supports fetching
 * options from a URL or a local data array. It features a standard input mode,
 * a read-only display mode, and an optional inline-editing capability for
 * read-only fields.
 *
 * @example
 * <!-- Populating with a local data array -->
 * <Select
 *   id="fruit-select"
 *   label="Favorite Fruit"
 *   v-model="selectedFruit"
 *   :data="fruitOptions"
 *   :errors="formErrors.fruit"
 * />
 *
 * <!-- Fetching options from a URL and enabling inline editing -->
 * <Select
 *   id="user-select"
 *   label="Assign User"
 *   v-model="assignedUser"
 *   url="/api/users"
 *   optionLabel="username"
 *   optionValue="id"
 *   :read-only="true"
 *   :allow-edit="true"
 * />
 */
import {capitalize} from "vue";

import FloatLabel from "primevue/floatlabel";
import PrimeSelect from 'primevue/select';

import apiCall from "@/util/Fetch";
import Toaster from "../util/Toaster.vue";
import InputText from "primevue/inputtext";
import X from "../icons/X.vue";
import Search from "../icons/Search.vue";
import Edit from "../icons/Edit.vue";
import Check from "../icons/Check.vue";

export default {
  name: "Select",
  /**
   * @property {String} [url] - URL to fetch the options list from.
   * @property {Array} [data] - An array of options to populate the dropdown.
   * @property {String} label - The label for the input, which will be translated.
   * @property {String} id - The unique identifier for the input element.
   * @property {String|Array|Number|Boolean} modelValue - The value of the component, for use with `v-model`.
   * @property {Object|Array} [errors] - Validation error messages.
   * @property {Boolean} [filter=false] - Whether to display a filter input in the dropdown.
   * @property {String} [optionValue="value"] - The property in the option object to use as the value.
   * @property {String} [optionLabel="name"] - The property in the option object to use as the label.
   * @property {Boolean} [readOnly=false] - If true, the component displays as text instead of a select field.
   * @property {Boolean} [allowEdit=false] - If true and in read-only mode, enables inline editing.
   */
  props: {
    url: String,
    data: Array,
    label: String,
    id: String,
    modelValue: {
      type: [String, Array, Number, Boolean],
      default: null
    },
    errors: [Object, Array],
    filter: {
      type: Boolean,
      default: false,
    },
    optionValue: {
      type: String,
      default: "value",
    },
    optionLabel: {
      type: String,
      default: "name",
    },
    readOnly: {
      type: Boolean,
      default: false,
    },
    allowEdit: {
      type: Boolean,
      default: false,
    },
  },
  /**
   * @event update:modelValue
   * @description Emitted to update the bound `v-model` value.
   * @type {String|Array|Number|Boolean}
   */
  emits: ['update:modelValue'],
  components: {
    Check,
    Edit,
    Search,
    X,
    InputText,
    FloatLabel,
    Toaster,
    PrimeSelect
  },
  data() {
    return {
      options: [],
      isLoading: !this.$props.data,
      isEditing: false,
      tmpVal: null,
    }
  },
  computed: {
    value: {
      get() {
        return this.modelValue;
      },
      set(newValue) {
        this.$emit('update:modelValue', newValue);
      }
    },
    selectedOptionText() {
      if (this.modelValue === null || this.modelValue === undefined) {
        return '';
      }
      const selected = this.options.find(item => item[this.optionValue] === this.modelValue);
      return selected ? selected[this.optionLabel] : '';
    }
  },
  methods: {
    capitalize,
    /**
     * Toggles the inline editing state.
     */
    toggleEdit() {
      this.isEditing = !this.isEditing;
    },
    /**
     * Discards changes made during inline editing.
     */
    dismissEdit() {
      this.tmpVal = this.modelValue;
      this.toggleEdit();
    },
    /**
     * Accepts the changes made during inline editing and emits an update.
     */
    acceptEdit() {
      this.$emit('update:modelValue', this.tmpVal);
      this.toggleEdit();
    },
  },
  watch: {
    modelValue: {
      handler(newValue) {
        this.$nextTick(() => {
          if (this.readOnly) {
            this.tmpVal = newValue;
            return;
          }
          // Workaround to ensure PrimeVue's FloatLabel updates correctly.
          const element = document.querySelector(`#${this.id}`);
          if (element) {
            element.dispatchEvent(new Event('input', {bubbles: true}));
          }
        });
      },
      immediate: true, // Run handler immediately on component mount
    }
  },
  async mounted() {
    if (this.$props.data) {
      this.options = this.$props.data;
      this.isLoading = false;
    } else if (this.$props.url) {
      try {
        this.options = await apiCall(this.url);
      } catch (e) {
        this.$refs.selectToasterRef.toast(
            "errors.messages.couldNotLoad",
            "errors.messages.load",
            "error",
            this.$props.label,
            5000,
        );
      } finally {
        this.isLoading = false;
      }
    }
  }
}
</script>

<template>
  <div v-if="!readOnly" class="row-element">
    <FloatLabel class="w-full md:w-56" variant="on">
      <PrimeSelect v-model="value"
                   :inputId="id"
                   :options="options"
                   class="w-full"
                   :filter="filter"
                   display="chip"
                   :optionValue="optionValue"
                   :optionLabel="optionLabel"
                   :invalid="!!errors"
                   style="width: 100%"
      />
      <label :for="id">{{ capitalize($t(label)) }}</label>
    </FloatLabel>
    <span class="error-text" v-if="errors">{{ $t(errors?.[0]) }}</span>
  </div>

  <div v-else class="readonly-element">
    <div>{{ capitalize($t(label)) }}</div>

    <div class="edit-row-element">
      <div v-if="!isEditing">{{ selectedOptionText }}</div>

      <div v-else class="inline-edit-input">
        <PrimeSelect v-model="tmpVal"
                     :inputId="id"
                     :options="options"
                     class="w-full"
                     :filter="filter"
                     display="chip"
                     :optionValue="optionValue"
                     :optionLabel="optionLabel"
                     :invalid="!!errors"
                     style="width: 100%"
        />
        <span v-if="errors" class="error-text">{{ $t(errors?.[0]) }}</span>
      </div>

      <div v-if="allowEdit" class="edit-controls">
        <template v-if="!isEditing">
          <!-- TODO: Edit this to be an svg or a component with an edit icon for use -->
          <Edit class="icon" @click="toggleEdit"/>
        </template>
        <template v-else>
          <!-- TODO: Edit this to be an svg or a component with a check icon for use -->
          <Check class="icon" @click="acceptEdit"/>
          <!-- TODO: Edit this to be an svg or a component with a dismiss icon for use -->
          <X class="icon" @click="dismissEdit"/>
        </template>
      </div>
    </div>
  </div>

  <Toaster ref="selectToasterRef"/>
</template>

<style scoped>
.readonly-element {
  display: grid;
  grid-template-columns: 1fr 2fr;
  grid-column-gap: 0.25rem;
  grid-row-gap: 0;
  min-height: 3rem;
  align-items: center;
}

.edit-row-element {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.edit-controls {
  margin-left: 0.25rem;
  margin-right: 0.25rem;
}

.icon:hover {
  cursor: pointer;
}
</style>

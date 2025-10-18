<script>
/**
 * @fileoverview A versatile text input component with modes for standard input,
 * read-only display, and inline editing.
 *
 * @description This component serves as a flexible input field. In its default
 * state, it's a standard text input with a floating label. When set to
 * read-only, it displays the value as plain text. Additionally, it supports an
 * inline editing feature in read-only mode, allowing users to modify the value
 * on the fly.
 *
 * @example
 * <!-- Basic usage with v-model and error display -->
 * <Input
 *   id="username"
 *   title="Username"
 *   v-model="formData.username"
 *   :errors="formErrors.username"
 * />
 *
 * <!-- Read-only display with inline editing enabled -->
 * <Input
 *   id="email"
 *   title="Email"
 *   v-model="userData.email"
 *   :read-only="true"
 *   :allow-edit="true"
 * />
 */
import {capitalize} from "vue";

import InputText from 'primevue/inputtext';
import FloatLabel from 'primevue/floatlabel';

import apiCall from "@/util/Fetch";
import Edit from "../icons/Edit.vue";
import X from "../icons/X.vue";
import Check from "../icons/Check.vue";

export default {
  name: "Input",
  /**
   * @property {String} id - The unique ID for the input element.
   * @property {String} title - The label text for the input, which will be translated.
   * @property {String | Array<Number>} modelValue - The binding for the input's value, used with `v-model`.
   * @property {Object | Array | null} errors - A list of validation errors to display.
   * @property {Boolean} [readOnly=false] - If true, the component displays as text instead of an input field.
   * @property {Boolean} [allowEdit=false] - If true and in read-only mode, enables inline editing.
   */
  props: {
    id: String,
    title: String,
    modelValue: String | Array(Number),
    errors: Object | Array | null,
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
   * @type {String | Array<Number>}
   */
  emits: ['update:modelValue'],
  components: {
    Check,
    X,
    Edit,
    InputText,
    FloatLabel,
    apiCall,
  },
  data() {
    return {
      isEditing: false,
      tmpVal: null, // Temporary storage for the value during inline editing
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
     * Discards changes made during inline editing and reverts to the original value.
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
      handler(_new, _old) {
        // Ensures the temporary value for inline editing is initialized.
        this.$nextTick(
            () => {
              if (this.readOnly) {
                this.tmpVal = this.modelValue;
                return;
              }
              // This is a workaround to ensure PrimeVue's FloatLabel updates correctly.
              document.querySelector(`#${this.id}`).dispatchEvent(new Event('input'));
            }
        );
      },
      once: true
    }
  },
}
</script>

<template>
  <!-- Standard Editable Input -->
  <div v-if="!readOnly" class="row-element">
    <FloatLabel variant="on">
      <InputText
          :id="id"
          :modelValue="modelValue"
          @update:modelValue="$emit('update:modelValue', $event)"
          :invalid="!!errors"
          style="width: 100%"
      />
      <label :for="id">{{ capitalize($t(title)) }}</label>
    </FloatLabel>
    <span v-if="errors" class="error-text">{{ $t(errors?.[0]) }}</span>
  </div>

  <!-- Read-Only Display with Optional Inline Editing -->
  <div v-else class="readonly-element">
    <div>{{ capitalize($t(title)) }}</div>

    <div class="edit-row-element">
      <!-- Display Value or Inline Edit Input -->
      <div v-if="!isEditing">{{ modelValue }}</div>
      <div v-else class="inline-edit-input">
        <InputText
            :id="id"
            v-model="tmpVal"
            :invalid="!!errors"
            style="width: 100%"
        />
        <span v-if="errors" class="error-text">{{ $t(errors?.[0]) }}</span>
      </div>

      <!-- Edit Controls (Toggle, Accept, Dismiss) -->
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

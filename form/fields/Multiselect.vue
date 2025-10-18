<script>
/**
 * @fileoverview A feature-rich multi-select dropdown component for forms.
 *
 * @description This component is a wrapper around PrimeVue's MultiSelect, providing a
 * standardized and enhanced way to handle multiple selections. It supports loading
 * options from a URL or a static data array, on-demand searching, and includes
 * actions for adding new items and clearing selections directly from the dropdown.
 * It also features a read-only mode with an optional inline-editing capability.
 *
 * Features:
 * - Binds to a model using `v-model`.
 * - Populates options from a `data` array or fetches them from a `url`.
 * - On-demand search functionality to load options as the user types (`onDemandSearch`).
 * - Footer buttons to add new items (`add`) or remove all selected items (`remove`).
 * - A simple dialog for adding new items (`simpleAdd`).
 * - A read-only mode to display selected items as a list.
 * - Inline-editing for read-only fields, enabled with `allowEdit`.
 * - Displays validation errors passed via the `errors` prop.
 *
 * @example
 * <!-- Example Usage -->
 * <Multiselect
 *   id="tags-select"
 *   label="Tags"
 *   v-model="selectedTags"
 *   :data="tagOptions"
 *   :errors="formErrors.tags"
 *   :add="true"
 *   :remove="true"
 * />
 *
 * <script>
 * export default {
 *   data() {
 *     return {
 *       selectedTags: [],
 *       tagOptions: [
 *         { name: 'Vue', id: 1 },
 *         { name: 'JavaScript', id: 2 },
 *         { name: 'PrimeVue', id: 3 },
 *       ],
 *       formErrors: {},
 *     };
 *   },
 * };
 * <'''>
 */
import {capitalize} from "vue";

import MultiSelect from 'primevue/multiselect';
import Button from 'primevue/button';
import FloatLabel from "primevue/floatlabel";
import ProgressBar from "primevue/progressbar";
import Dialog from "primevue/dialog";

import apiCall from "@/util/Fetch";
import Add from "@/components/icons/Add.vue";
import Toaster from "@/components/util/Toaster.vue";
import Remove from "../icons/Remove.vue";
import Input from "./Input.vue";
import LoadingDialog from "../modal/Loader.vue";
import InputText from "primevue/inputtext";
import {useI18n} from "vue-i18n";
import X from "../icons/X.vue";
import Check from "../icons/Check.vue";
import Edit from "../icons/Edit.vue";

export default {
  name: "Multiselect",
  methods: {
    capitalize,
    addEntry() {
      this.showAddDialog = true;
    },
    removeAll() {
      if (this.readOnly) {
        this.tmpVal = [];
        return;
      }
      this.$emit('update:modelValue', []);
    },
    async createElement() {
      try {
        this.showCreateDialog = true;

        const url = this.addUrl || this.url;
        const body = {
          name: this.addInputValue,
        };
        await apiCall(url, "POST", body);

        this.showAddDialog = false;
        this.addInputValue = '';
        this.remount = !this.remount;
        this.$refs.multiselect.toggle();
      } catch (e) {
        this.$refs.multiselectToasterRef.toast(
            "errors.messages.couldNotCreate",
            "errors.messages.load",
            "error",
            this.$props.label,
            5000,
        );
      } finally {
        this.showCreateDialog = false;
      }
    },
    onFilter(event) {
      if (!this.$props.onDemandSearch) {
        return;
      }
      this.filterValue = event.value;
    },
    toggleEdit() {
      this.isEditing = !this.isEditing;
    },
    dismissEdit() {
      this.tmpVal = this.modelValue;
      this.toggleEdit();
    },
    acceptEdit() {
      this.$emit('update:modelValue', this.tmpVal);
      this.toggleEdit();
    },
  },
  /**
   * @property {String} [url] - URL to fetch the options list from.
   * @property {String} [addUrl] - A specific URL for adding new items. Defaults to `url`.
   * @property {Array} [data] - An array of options to populate the dropdown.
   * @property {String} label - The label for the input, which will be translated.
   * @property {String} id - The unique identifier for the input element.
   * @property {Boolean} [simpleAdd=true] - If true, a simple dialog is used for adding new items.
   * @property {Array} modelValue - The value of the component, for use with `v-model`.
   * @property {Object|Array} [errors] - Validation error messages.
   * @property {Boolean} [add=false] - If true, shows an "Add" button in the dropdown footer.
   * @property {Boolean} [remove=false] - If true, shows a "Remove All" button in the dropdown footer.
   * @property {Boolean} [onDemandSearch=false] - If true, options are fetched as the user types.
   * @property {String} [emptyMessage='no entries'] - Message to display when no options are available.
   * @property {String} [emptyFilterMessage='no entries'] - Message to display when a filter returns no results.
   * @property {Boolean} [readOnly=false] - If true, the component displays as text instead of a multiselect field.
   * @property {Boolean} [allowEdit=false] - If true and in read-only mode, enables inline editing.
   * @property {String} [readOnlyDataKey="name"] - The property to display for selected items in read-only mode.
   */
  props: {
    url: String,
    addUrl: String | null,
    data: Array,
    label: String,
    id: String,
    simpleAdd: {
      type: Boolean,
      default: true,
    },
    modelValue: {
      type: Array,
      default: () => []
    },
    errors: [Object, Array],
    add: {
      type: Boolean,
      default: false,
    },
    remove: {
      type: Boolean,
      default: false,
    },
    onDemandSearch: {
      type: Boolean,
      default: false,
    },
    emptyMessage: {
      type: String,
      default: 'no entries',
    },
    emptyFilterMessage: {
      type: String,
      default: 'no entries',
    },
    readOnly: {
      type: Boolean,
      default: false,
    },
    allowEdit: {
      type: Boolean,
      default: false,
    },
    readOnlyDataKey: {
      type: String,
      default: "name"
    }
  },
  components: {
    Edit,
    Check,
    X,
    InputText,
    LoadingDialog,
    Input,
    Dialog,
    ProgressBar,
    Remove,
    FloatLabel,
    Toaster,
    MultiSelect,
    Add,
    Button,
  },
  data() {
    return {
      options: [],
      isLoading: !this.$props.data && !this.$props.onDemandSearch,
      showAddDialog: false,
      addInputValue: '',
      remount: false,
      showCreateDialog: false,
      filterValue: '',
      isEditing: false,
      tmpVal: null
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
    loaderText() {
      return `creating.loading.${this.label}`;
    },
    selectedItemsLabelText() {
      const {t} = useI18n();

      const selectValue = this.readOnly ? this.tmpVal : this.modelValue;
      const count = selectValue ? selectValue.length : 0;
      const baseText = t('multipleSelected');
      const label = `${count} ${capitalize(baseText)}`;

      return label;
    }
  },
  watch: {
    modelValue: {
      handler(newValue, _old) {
        this.$nextTick(
            () => {
              if (this.readOnly) {
                this.tmpVal = newValue;
                return;
              }
              // Dispatch an input event to ensure form libraries that listen for it can react.
              const element = document.querySelector(`#${this.id}`);
              if(element) {
                element.dispatchEvent(new Event('input', { bubbles: true }));
              }
            }
        );
      },
      immediate: true,
      deep: true
    },
    tmpVal: {
      handler(newValue, _old) {
        this.$nextTick(
            () => {
              if (this.readOnly && this.onDemandSearch && newValue) {
                this.options = newValue;
              }
            }
        );
      },
      immediate: true,
      deep: true
    },
    async filterValue(_new, _old) {
      if (_old && _old.length >= 2 && _new.length < 2) {
        this.options = this.value || [];
        return;
      }

      if (_new && _new.length >= 2) {
        try {
          const search = {search: _new};
          this.isLoading = true;
          let fetchedOptions = await apiCall(this.url, "GET", null, null, search);

          if (this.value) {
            // Combine fetched options with already selected items
            const combined = [...fetchedOptions, ...this.value];
            // Filter out duplicates
            this.options = combined.filter((obj, index, self) =>
                index === self.findIndex((item) => item.id === obj.id)
            );
          } else {
            this.options = fetchedOptions;
          }
        } catch (e) {
          this.$refs.multiselectToasterRef.toast(
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
    },
  },
  async mounted() {
    // Initialize tmpVal for inline editing
    if (this.readOnly) {
      this.tmpVal = this.modelValue;
    }

    if (this.$props.onDemandSearch) {
      this.options = this.value || [];
      this.isLoading = false;
      return;
    }
    if (this.$props.data) {
      this.options = this.$props.data;
      this.isLoading = false;
      return;
    }
    if (this.$props.url) {
      try {
        this.isLoading = true;
        this.options = await apiCall(this.url);
      } catch (e) {
        this.$refs.multiselectToasterRef.toast(
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
  },
  async beforeUpdate() {
    if (!this.$props.data && this.value && this.isLoading && !this.onDemandSearch) {
      this.options = this.value;
    }
  }
}
</script>

<template>
  <div :key="remount" class="row-element" v-if="!readOnly">
    <FloatLabel class="w-full md:w-80" variant="on">
      <MultiSelect v-model="value"
                   :id="id"
                   :options="options"
                   :loading="isLoading"
                   :emptyMessage="capitalize($t(emptyMessage))"
                   :emptyFilterMessage="capitalize($t(emptyFilterMessage))"
                   optionLabel="name"
                   filter
                   @filter="onFilter"
                   display="chip"
                   class="w-full md:w-80 input"
                   ref="multiselect"
                   :invalid="!!errors"
                   :maxSelectedLabels="3"
                   :selectedItemsLabel="selectedItemsLabelText"
                   style="width: 100%"
      >
        <template #option="slotProps">
          <div style="margin-bottom: 0.31rem">
            {{ slotProps.option.name }}
          </div>
        </template>
        <template #footer>
          <div class="buttons">
            <Button severity="success" variant="outlined" size="small" v-if="add" @click="addEntry">
              <Add/>
              <span>{{ capitalize($t('add')) }}</span>
            </Button>
            <Button severity="danger" variant="outlined" size="small" v-if="remove" @click="removeAll">
              <Remove/>
              <span>{{ capitalize($t('remove all')) }}</span>
            </Button>
          </div>
        </template>
      </MultiSelect>
      <label class="label" :for="id">{{ capitalize($t(label)) }}</label>
    </FloatLabel>
    <span class="error-text" v-if="errors">{{ $t(errors?.[0]) }}</span>
  </div>
  <div v-else class="readonly-element">
    <div>{{ capitalize($t(label)) }}</div>
    <div class="edit-row-element">
      <div v-if="!isEditing">
        <div v-for="item in modelValue" :key="item.id">
          - {{ item[readOnlyDataKey] }}
        </div>
      </div>

      <div v-else class="inline-edit-input">
        <MultiSelect v-model="tmpVal"
                     :id="id"
                     :options="options"
                     :loading="isLoading"
                     :emptyMessage="capitalize($t(emptyMessage))"
                     :emptyFilterMessage="capitalize($t(emptyFilterMessage))"
                     optionLabel="name"
                     filter
                     @filter="onFilter"
                     display="chip"
                     class="w-full md:w-80 input"
                     ref="multiselect"
                     :invalid="!!errors"
                     :maxSelectedLabels="0"
                     :selectedItemsLabel="selectedItemsLabelText"
        >
          <template #option="slotProps">
            <div style="margin-bottom: 0.31rem">
              {{ slotProps.option.name }}
            </div>
          </template>
          <template #footer>
            <div class="buttons">
              <Button severity="success" variant="outlined" size="small" v-if="add" @click="addEntry">
                <Add/>
                <span>{{ capitalize($t('add')) }}</span>
              </Button>
              <Button severity="danger" variant="outlined" size="small" v-if="remove" @click="removeAll">
                <Remove/>
                <span>{{ capitalize($t('remove all')) }}</span>
              </Button>
            </div>
          </template>
        </MultiSelect>
        <span v-if="errors" class="error-text">{{ $t(errors?.[0]) }}</span>
      </div>
      <div v-if="allowEdit" class="edit-controls">
        <template v-if="!isEditing">
          <!-- TODO: Edit this to be an svg or a component with an edit icon for use -->
          <Edit class="icon" @click="toggleEdit"/>
        </template>
        <template v-else>
          <div class="edit-actions">
          <!-- TODO: Edit this to be an svg or a component with a check icon for use -->
            <Check class="icon" @click="acceptEdit"/>
          <!-- TODO: Edit this to be an svg or a component with a dismiss icon for use -->
            <X class="icon" @click="dismissEdit"/>
          </div>
        </template>
      </div>
    </div>
  </div>
  <Toaster ref="multiselectToasterRef"/>

  <Dialog
      :visible="showAddDialog"
      :modal="true"
      :closable="false"
      :style="{ width: '50vw' }"
      :draggable="false"
      :resizable="false"
      v-if="add && simpleAdd"
  >
    <template #header>
      <div class="centered-header">
        <span class="p-dialog-title">{{ capitalize($t(label)) }} {{ $t('add') }}</span>
      </div>
    </template>
    <div class="dialog-content">
      <Input id="addInput" :title="label" v-model.trim="addInputValue"/>
    </div>
    <template #footer>
      <Button severity="secondary" style="margin-right: 0.5rem" @click="() => {showAddDialog = false}">
        <span>{{ capitalize($t('cancel')) }}</span>
      </Button>
      <Button severity="success" @click="createElement">
        <span>{{ capitalize($t('add')) }}</span>
      </Button>
    </template>
  </Dialog>

  <LoadingDialog :visible="showCreateDialog" :header="loaderText"/>
</template>

<style scoped>
.buttons {
  display: flex;
  padding: 0.5rem;
  justify-content: center;
  flex-direction: column;
}

.buttons > * {
  margin-bottom: 0.25rem;
  padding-top: 0.1rem;
  padding-bottom: 0.1rem;
}

.dialog-content {
  margin-top: 0.5rem;
}

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
  overflow-x: hidden;
  min-width: 0;
}

.edit-controls {
  margin-left: 0.25rem;
  margin-right: 0.25rem;
}

.icon:hover {
  cursor: pointer;
}

.inline-edit-input {
  box-sizing: border-box;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
</style>

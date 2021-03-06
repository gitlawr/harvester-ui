<script>
/* eslint-disable */
import { clone } from '@/utils/object';
import VMModal from '@/components/form/VMModal';
import LabeledInput from '@/components/form/LabeledInput';
import LabeledSelect from '@/components/form/LabeledSelect';
import { sortBy } from '@/utils/sort';
import { NETWORK_ATTACHMENT } from '@/config/types';

export default {
  components: {
    VMModal,
    LabeledInput,
    LabeledSelect
  },

  props: {
    value: {
      type:    Array,
      default: () => {
        return {};
      }
    },
    namespace: {
      type: String,
      default: ''
    },
    rowActions: {
      type:    Boolean,
      default: true
    }
  },

  data() {
    return {
      rows:       clone(this.value),
      type:       'add',
      rowIndex:   0,
      errors:     [],
      currentRow: {}
    };
  },

  computed: {
    headers() {
      return [{
        name:     'name',
        label:    'Name',
        value:    'name',
      },
      {
        name:     'model',
        label:    'Model',
        value:    'model',
      },
      {
        name:     'network',
        label:    'Network',
        value:    'networkName',
      },
      {
        name:     'type',
        label:    'Type',
        value:    'type',
      },
      {
        name:     'macAddress',
        label:    'MAC Address',
        value:    'macAddress',
      }];
    },

    modelOption() {
      return [{
        label: 'virtio',
        value: 'virtio'
      },
      {
        label: 'e1000',
        value: 'e1000'
      },
      {
        label: 'e1000e',
        value: 'e1000e'
      },
      {
        label: 'ne2k_pci',
        value: 'ne2k_pci'
      },
      {
        label: 'pcnet',
        value: 'pcnet'
      },
      {
        label: 'rtl8139',
        value: 'rtl8139'
      }];
    },

    networkOption() {
      const choices = this.$store.getters['cluster/all'](NETWORK_ATTACHMENT);

      const out = sortBy(
        choices
          .filter((C) => C.metadata.namespace === this.namespace)
          .map((obj) => {
            return {
              label: obj.metadata.name,
              value: obj.metadata.name
            };
          }),
        'label'
      );

      const findPodIndex = _.findIndex(this.rows, (o) => {
        return o.networkName === 'Pod Network'
      })
      if (findPodIndex === -1 || (findPodIndex !== -1 && this.rows.length ===1 && this.type !== 'add' )) {
        out.push({
          label: 'Pod Network',
          value: 'Pod Network'
        })
      }

      return out;
    },

    isMasquerade() {
      return this.currentRow.networkName === 'Pod Network'
    },

    typeOpton() {
      if (this.currentRow.networkName === 'Pod Network') {
        return [{
          label: 'masquerade',
          value: 'masquerade'
        }, {
          label: 'bridge',
          value: 'bridge'
        }]
      }

      return [{
        label: 'bridge',
        value: 'bridge'
      },
      {
        label: 'sriov',
        value: 'sriov'
      }];
    },
  },

  watch: {
    value(neu) {
      this.rows = neu;
    },
    'currentRow.networkName'(neu) {
      if (neu === 'Pod Network') {
        this.currentRow.type = 'masquerade'
      } else {
        this.currentRow.type = 'bridge'
      }
    }
  },

  methods: {
    updateAdd() {
      if (this.type === 'add') {
        this.rows.splice(this.rowIndex, 0, this.currentRow);
      } else if (this.type === 'delete') {
        this.rows.splice(this.rowIndex, 1);
      } else {
        this.rows.splice(this.rowIndex, 1, this.currentRow);
      }

      this.rows.forEach((o, index) => {
        o.index = index;
      });

      this.$emit('input', this.rows);
    },
    beforeCancel() {
      this.$set(this, 'errors', []);
    },
    updateIndex(index, type) {
      this.rowIndex = index;
      this.type = type;
      const networkName = this.networkOption?.[0]?.value || '';

      this.currentRow = clone(this.rows[this.rowIndex]) || {
        name: `nic-${ index }`, model: 'virtio', networkName, type: 'bridge'
      };
    },

    validateError() {
      if (
        this.currentRow.model &&
        this.currentRow.name &&
        this.currentRow.networkName &&
        this.currentRow.type
      ) {
        this.$set(this, 'errors', []);
      } else {
        this.errors.splice(0, 1, 'Please fill in all required fields.');
      }
    },

    validateMac(value) {
      if (!/^[A-F0-9]{2}(-[A-F0-9]{2}){5}$|^[A-F0-9]{2}(:[A-F0-9]{2}){5}$/.test(value)) {
        this.errors.splice(0, 1, 'Invalid MAC address format.');
      } else {
        this.$set(this, 'errors', []);
      }
    }
  }
};
</script>

<template>
  <div>
    <VMModal
      :row-actions="rowActions"
      modal-name="network"
      title="Add Network Interface"
      :rows="rows"
      :headers="headers"
      :errors="errors"
      @update:cancel="beforeCancel"
      @update:add="updateAdd"
      @update:index="updateIndex"
      @validateError="validateError"
    >
      <template v-slot:content>
        <LabeledInput
          v-model="currentRow.name"
          label="Name"
          class="mb-20"
          required
        />

        <LabeledSelect
          v-model="currentRow.model"
          label="Model"
          :options="modelOption"
          class="mb-20"
          required
        />

        <LabeledSelect
          v-model="currentRow.networkName"
          label="Network"
          :options="networkOption"
          class="mb-20"
          required
        />

        <LabeledSelect
          v-model="currentRow.type"
          label="Type"
          :options="typeOpton"
          class="mb-20"
          required
        />

        <LabeledInput
          v-model="currentRow.macAddress"
          label="Mac Address"
          v-if="!isMasquerade"
          @input="validateMac"
        />
        <h5 class="tip" v-if="!isMasquerade">
          Protip: MAC address as seen inside the guest system.
        </h5>
      </template>
    </VMModal>
  </div>
</template>

<style lang="scss" scoped>
.tip {
  font-size: 13px;
  font-style: italic;
}
</style>

<script>
import { VMI } from '@/config/types';
import Console from '@/components/form/Console';

export default {
  layout: 'blank',

  components: { Console },

  data() {
    return { uid: this.$route.params.uid };
  },

  computed: {
    vmi() {
      const vmiList = this.$store.getters['cluster/all'](VMI) || [];
      const vmi = vmiList.find( (VMI) => {
        return VMI?.metadata?.ownerReferences?.[0]?.uid === this.uid;
      });

      return vmi;
    },
  },

  mounted() {
    window.addEventListener('beforeunload', () => {
      this.$refs.console.close();
    });
  },
};
</script>

<template>
  <Console ref="console" v-model="vmi" />
</template>

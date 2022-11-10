<template>
  <el-dialog :show-close="false" :visible="open" @close="closeHandler" class="json-forms-dialog">
    <bf-dialog-header slot="title" title="Inputs demo" />
    <form class="osparc-request-form" @submit="fetchOsparc">
      <input name="service" value="simcore/services/comp/itis/sleeper"/>
      <input name="version" value="2.1.1"/>
      <button type="submit">Fetch ports</button>
    </form>
    <dialog-body>
      <json-forms
        v-if="schema !== null"
        class="json-form"
        :schema="schema"
        :data="data"
        :renderers="renderers"
      />
    </dialog-body>
  </el-dialog>
</template>

<script>
import BfDialogHeader from '@/components/bf-dialog-header/BfDialogHeader.vue'
import DialogBody from '@/components/dialog-body/DialogBody.vue'
import { JsonForms } from '@jsonforms/vue2'
import { vanillaRenderers } from '@jsonforms/vue2-vanilla'

const renderers = [
  ...vanillaRenderers
]

const data = {}

const generateSchemaFromInputs = inputs => {
  if (Array.isArray(inputs)) {
    const properties = {}
    inputs.forEach(input => {
      properties[input.key] = input['content_schema']
    })
    return {
      type: 'object',
      properties
    }
  }
}

export default {
  name: 'ComputationalInputsDialog',
  components: { BfDialogHeader, DialogBody, JsonForms },
  props: {
    open: {
      type: Boolean,
    },
  },
  data: () => ({
    schema: null,
    data,
    renderers: Object.freeze(renderers)
  }),
  methods: {
    closeHandler() {
      this.$emit('close')
    },
    onChange(e) {
      this.data = e.data
    },
    fetchOsparc(e) {
      e.preventDefault()
      const formData = new FormData(e.target)
      const url = new URL(`${process.env['portal_api']}/get-service-inputs`)
      url.searchParams.append('service', formData.get('service'))
      url.searchParams.append('version', formData.get('version'))
      fetch(url)
        .then(resp => resp.json())
        .then(data => {
          this.schema = generateSchemaFromInputs(data.inputs)
        })
    }
  }
}
</script>
<style lang="scss" scoped>
.json-forms-dialog {
  text-align: left;
}
.osparc-request-form {
  margin-bottom: 20px;
  & > * {
    display: block;
    margin-bottom: 10px;
  }
}
</style>
<style lang="scss">
.json-form {
  & .control {
    margin-bottom: 20px;
  }
}
</style>

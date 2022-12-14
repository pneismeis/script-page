<template>
  <div class="script-parameters-panel">
    <template v-for="parameter in parameters">
      <component
          :is="getComponentType(parameter)"
          :key="parameter.name"
          :config="parameter"
          :value="parameterValues[parameter.name]"
          :class="{'inline': isInline(parameter)}"
          class="parameter"
          :forceValue="forcedValueParameters.includes(parameter.name)"
          @error="handleError(parameter, $event)"
          @input="setParameterValue(parameter.name, $event)"/>
    </template>
  </div>
</template>

<script>
import Checkbox from '@/common/components/checkbox'
import Combobox from '@/common/components/combobox'
import FileUpload from '@/common/components/file_upload'
import ServerFileField from '@/common/components/server_file_field'
import Textfield from '@/common/components/textfield'
import {mapActions, mapState} from 'vuex'
import {comboboxTypes, isRecursiveFileParameter} from '../../utils/model_helper'
import TextArea from '@/common/components/TextArea'

export default {
  name: 'script-parameters-view',

  computed: {
    ...mapState('scriptConfig', {
      parameters: 'parameters'
    }),
    ...mapState('scriptSetup', {
      parameterValues: 'parameterValues',
      forcedValueParameters: 'forcedValueParameters',
    })
  },

  methods: {
    ...mapActions('scriptSetup', {
      setParameterValueInStore: 'setParameterValue',
      setParameterErrorInStore: 'setParameterError'
    }),
    getComponentType(parameter) {
      if (parameter.withoutValue) {
        return Checkbox;
      } else if (isRecursiveFileParameter(parameter)) {
        return ServerFileField;
      } else if (comboboxTypes.includes(parameter.type)) {
        return Combobox;
      } else if (parameter.type === 'file_upload') {
        return FileUpload;
      } else if (parameter.type === 'multiline_text') {
        return TextArea;
      } else {
        return Textfield;
      }
    },

    isInline(parameter) {
      return parameter.type !== 'multiline_text'
    },

    handleError(parameter, error) {
      this.setParameterErrorInStore({parameterName: parameter.name, errorMessage: error})
    },

    setParameterValue(parameterName, value) {
      this.setParameterValueInStore({parameterName, value});
    }
  }
}
</script>

<style scoped>
.script-parameters-panel >>> {
  margin-top: 15px;
  margin-right: 0;
  display: flex;
  flex-wrap: wrap;
}

.script-parameters-panel >>> .parameter {
  margin: 7px 24px 20px 0;

  flex-grow: 1;
  flex-shrink: 0;
  width: 180px;
  max-width: 220px;
}

.script-parameters-panel >>> .parameter input,
.script-parameters-panel >>> .parameter textarea,
.script-parameters-panel >>> .parameter .file-upload-field-value {
  margin: 0;

  font-size: 1rem;
  height: 1.5em;
  line-height: 1.5em;
}

.script-parameters-panel >>> .parameter:not(.inline) {
  flex-basis: 100%;
  max-width: 100%;
}

.script-parameters-panel >>> .parameter textarea {
  padding-bottom: 0;
  padding-top: 0;

  min-height: 1.5em;
  box-sizing: content-box;
}

.script-parameters-panel >>> .parameter > label {
  transform: none;
  font-size: 1rem;
}

.script-parameters-panel >>> .parameter > label.active {
  transform: translateY(-70%) scale(0.8);
}

.script-parameters-panel >>> .input-field input[type=checkbox] + span {
  padding-left: 28px;
}

.script-parameters-panel >>> .input-field .select-wrapper + label {
  transform: scale(0.8);
  top: -18px;
}

.script-parameters-panel >>> .input-field:after {
  top: 1.7em;
  left: 0.1em;
}

.script-parameters-panel >>> .file-upload-field .btn-icon-flat {
  top: -7px;
  right: -4px;
}

.script-parameters-panel >>> .dropdown-content {
  max-width: 50vw;
  min-width: 100%;
  white-space: nowrap;

  margin-bottom: 0;
}

.script-parameters-panel >>> .dropdown-content > li > span {
  overflow-x: hidden;
  text-overflow: ellipsis;
}

</style>

<template >
  <div>
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
      <a class="navbar-brand" href="#">{{ title }}</a>
      <el-button
        type="primary"
        icon="el-icon-plus"
        @click="issueFormActive = true">Issue</el-button>
      <el-button
        type="primary"
        icon="el-icon-refresh"
        @click="$emit('issue-cred-refresh',)"></el-button>
    </nav>
    <el-collapse v-model="expanded_items">
      <ul class="list">
        <el-collapse-item
          v-for="issued_credential in credentials"
          v-bind:title="credential_title(issued_credential)"
          :name="issued_credential.credential_exchange_id"
          :key="issued_credential.credential_exchange_id">
          <el-row>
            <div>
              <vue-json-pretty
                :deep=1
                :data="issued_credential">
              </vue-json-pretty>
            </div>
            <el-button v-on:click="collapse_expanded(issued_credential)">^</el-button>
          </el-row>
        </el-collapse-item>
      </ul>
    </el-collapse>
    <el-dialog title="Issue Credential" :visible.sync="issueFormActive" @close="deActivateForm()">
      <el-form :model="issueForm">
        <el-form-item label="Connection:" :label-width="formLabelWidth">
          <el-select
            v-model="issueForm.connection_id"
            filterable
            value-key="issueForm.connection_id"
            placeholder="Select">
            <el-option
              v-for="connection in connections"
              :key="connection.connection_id"
              :label="connection.label"
              :value="connection.connection_id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="Credential Definition:" :label-width="formLabelWidth">
          <el-select
            v-model="issueForm.selected_cred_def"
            remote
            value-key="issueForm.selected_cred_def"
            placeholder="Select"
            :disabled="!issueForm.connection_id"
            @change="update_attributes">
            <el-option
              v-for="cred_def in cred_defs"
              :key="cred_def.cred_def_id"
              :label="cred_def.cred_def_id"
              :value="cred_def">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item
          label="Comment (optional)"
          :label-width="formLabelWidth">
          <el-input
            v-model="issueForm.comment"
            type="textarea"></el-input>
        </el-form-item>
        <el-form-item
          v-for="attribute in issueForm.attributes"
          :label="attribute.name"
          :label-width="formLabelWidth"
          :key="attribute.name">
          <el-input
            v-model="attribute.value"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="deActivateForm()">Cancel</el-button>
        <el-button :disabled="!issueForm.selected_cred_def" type="primary" @click="issue">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import VueJsonPretty from 'vue-json-pretty';
import share from '@/share.js';

export default {
  name: 'issued-cred-list',
  props: ['title', 'list', 'connections', 'cred_defs'],
  mixins: [share({use: ['id_to_connection']})],
  components: {
    VueJsonPretty,
  },
  data () {
    return {
      expanded_items:[],
      issueFormActive: false,
      issueForm: {
        connection_id: '',
        selected_cred_def: null,
        comment: '',
        attributes: []
      },
      retrieve_cred_def_id: '',
      formLabelWidth: '200px'
    }
  },
  computed: {
    credentials: function() {
      return this.list.map(item => {
        if (item.connection_id in this.id_to_connection) {
          item.connection = this.id_to_connection[item.connection_id];
        } else {
          item.connection = null;
        }
        return item;
      });
    },
  },
  methods: {
    collapse_expanded: function(creddef) {
      this.expanded_items = this.expanded_items.filter(
        item => item != creddef.credential_exchange_id
      );
    },
    deActivateForm: function() {
      this.issueFormActive = false;
      this.issueForm = {
        connection_id: '',
        selected_cred_def: null,
        comment: '',
        attributes: []
      };
    },
    issue: function() {
      let values = {
        connection_id: this.issueForm.connection_id,
        cred_def_id: this.issueForm.selected_cred_def.cred_def_id,
        comment: this.issueForm.comment,
        attributes: this.issueForm.attributes,
      }
      this.issueForm.connection_id = '';
      this.issueForm.selected_cred_def = null;
      this.issueForm.comment = '';
      this.issueForm.attributes = [];

      this.$emit('issue', values);
      this.issueFormActive = false;
    },
    update_attributes: function(cred_def) {
      var comp = this;
      comp.issueForm.attributes = [];
      cred_def.attributes.forEach(name => {
        comp.issueForm.attributes.push({
          name: name,
          value: ''
        });
      });
    },
    credential_title: function(cred) {
      let split = cred.schema_id.split(':');
      let connection_name = '';
      if (!cred.connection) {
        connection_name = '[deleted]';
      } else {
        connection_name = cred.connection.label;
      }
      return `${split[2]} v${split[3]} issued to ${connection_name}`;
    }
  }
}
</script>

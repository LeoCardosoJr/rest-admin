<template>
  <div v-if="model">
    <form ref="form" :action="actionUrl" :method="method" class="form form-inline" @submit.prevent="handleSubmit" v-if="inline" enctype="multipart/form-data">
      <input type="hidden" name="token" :value="auth.token">

      <template v-for="(field, name) in fields">
        <label :for="'input_' + name" class="m-1" :key="name">{{field.label || $inflection.titleize(name)}}</label>
        <b-form-field :languages="languages" :parent="model" class="m-1 mr-4" v-model="model[name]" :id="'input_' + name" :name="name" :field="field" :state="!hasError(name)" :key="id + '_' +name" />
      </template>

      <slot name="actions">
        <b-button type="submit" variant="primary" ref="submitButton">{{submitText}}</b-button>
        <b-button type="button" variant="secondary" @click="$router.go(-1)" v-if="backText">{{backText}}</b-button>
        <slot name="extra-buttons"></slot>
      </slot>
    </form>

    

    <form ref="form" :action="actionUrl" :method="method"  class="form" @submit.prevent="handleSubmit" enctype="multipart/form-data" v-else>
      <input type="hidden" name="token" :value="auth.token">
      <b-tabs class="my-3" v-if="groupBy">
        <b-tab v-for="(subFields, tabName) in groupedFields" :title="tabName || $t('messages.default')" :key="tabName">
          <div class="row">
            <b-col :md="_.get(layout, `tabs.${tabName}.cols`, 12)">
              <b-row>
                <b-form-group :class="getClass(field)"  v-if="isShowField(field) && model" :state="!hasError(name)" 
            v-for="(field, name) in subFields" :key="id + '_' +name" v-bind="field" :label-for="'input_' + name"
            :label="field.label || $inflection.titleize(name)">
              <div class="">
                <b-form-field :languages="languages" :class="getInputClass(field)" :parent="model" 
                @input="setValue(name, arguments[0], arguments[1])" :value="model[name]"
                :name="name" :field="field" :state="!hasError(name)" :id="'input_' + name" />
              </div>
            </b-form-group>
              </b-row>
            </b-col>
            <b-col :md="12 - _.get(layout, `tabs.${tabName}.cols`, 0)">
              <div v-html="_.get(layout, `tabs.${tabName}.right`)"></div>
            </b-col>
          </div>
        </b-tab>
      </b-tabs>
      <div class="row" v-else>
        <b-form-group :class="getClass(field)"  v-if="isShowField(field) && model" :state="!hasError(name)" 
        v-for="(field, name) in fields" :key="id + '_' +name" v-bind="field" :label-for="'input_' + name"
        :label="field.label || $inflection.titleize(name)">
          <div class="">
            <b-form-field :languages="languages" :class="getInputClass(field)" :parent="model" v-model="model[name]" :name="name" :field="field" :state="!hasError(name)" :id="'input_' + name" />
          </div>
        </b-form-group>
      </div>
      

      <slot name="actions">
        <b-button type="submit" variant="primary" ref="submitButton">{{submitText}}</b-button>
        <b-button type="button" variant="secondary" @click="$router.go(-1)" v-if="backText">{{backText}}</b-button>
      </slot>
    </form>
  </div>
</template>

<script>
import _ from "lodash";

export default {
  name: "b-form-builder",
  components: {},
  props: {
    id: {
      type: String,
      default() {
        return "form_" + parseInt(Math.random() * 9999);
      }
    },
    auth: {
      type: Object,
      default() {
        return {};
      }
    },
    col: {
      type: Number,
      default: 12
    },
    languages: {},
    inline: {
      type: Boolean,
      default: false
    },
    useFormData: {
      type: Boolean,
      default: false
    },
    submitRawForm: {
      type: Boolean,
      default: false
    },
    groupBy: {
      type: String,
      default: null
    },
    value: {
      default() {
        return {};
      }
    },
    fields: {
      required: true,
      default() {
        return {};
      }
    },
    layout: {
      required: false,
      default() {
        return {};
      }
    },
    onSubmit: {
      type: Function,
      required: false
    },
    action: {},
    method: {
      default: "post"
    },
    submitText: {
      default() {
        return this.$t ? this.$t("actions.save") : "Submit";
      }
    },
    backText: {
      default() {
        return this.$t ? this.$t("actions.back") : "Back";
      }
    },

    successMessage: {
      default() {
        return this.$t ? this.$t("messages.succeed") : "Succeed";
      }
    }
  },
  data() {
    return {
      model: null,
      errors: []
    };
  },
  watch: {
    value(val) {
      this.model = val;
    }
  },
  computed: {
    actionUrl() {
      return global.API_URI + this.action;
    },
    groupedFields() {
      const ret = {};
      _.keys(_.groupBy(this.fields, this.groupBy)).map(v => {
        // console.log(v, v == 'undefined')
        let tabName = v;
        if (v == "undefined") {
          v = null;
          tabName = this.$t("messages.default");
        }
        ret[tabName] = _.pickBy(this.fields, field => field.group == v);
      });
      return ret;
    }
  },
  methods: {
    setValue(name, value, lang) {
      if (!this.fields[name].multilingual) {
        return this.$set(this.model, name, value)
        // return _.set(this.model, name, value);
      }
      if (!this.model[name] && lang) {
        this.$set(this.model, name, {});
      }
      this.$set(this.model[name], lang, value)
      // _.set(this.model, key, value);
      // console.log(key, value)
    },
    titlize() {},
    isShowField(field) {
      return (
        !field.showWhen || this.model[field.showWhen] || eval(field.showWhen)
      );
    },
    getInputClass() {
      return [];
      // const classNames = [];
      // classNames.push(`col-lg-${field.input_cols ? field.input_cols : "12"}`);
      // return classNames;
    },
    getClass(field) {
      const cols = field.cols ? field.cols : 12;
      const classNames = ["col-lg-" + cols, "col-" + Math.min(12, cols * 2)];
      return classNames;
    },
    hasError(name) {
      return _.find(this.errors, v => v.field == name);
    },
    submitForm() {
      this.$refs.submitButton.click();
    },
    handleSubmit() {
      if (this.submitRawForm) {
        this.$refs.form.submit();
        return true;
      }
      if (this.onSubmit) {
        return this.onSubmit(this.model);
      }
      const methodName = String(this.method).toLowerCase();
      // console.log(this.$refs.form);
      let formData = this.model;
      if (this.useFormData) {
        formData = new FormData();
        _.mapValues(this.model, (v, k) => formData.append(k, v));
      }
      this.$http[methodName](this.action, formData)
        .then(({ data }) => {
          if (this.successMessage) {
            this.$snotify.success(this.successMessage);
          }
          this.errors = [];
          this.$emit("success", data);
        })
        .catch(({ data, status }) => {
          if (status == 422) {
            this.errors = data.message;
          }
        });
    }
  },
  mounted() {
    this.model = this.value;
    for (let [k, v] of Object.entries(this.fields)) {
      if (v.type === "object" && !this.model[k]) {
        this.$set(this.model, k, {});
      }
    }
    // global.console.log(this.fields, this.model)
  },
  created() {}
};
</script>


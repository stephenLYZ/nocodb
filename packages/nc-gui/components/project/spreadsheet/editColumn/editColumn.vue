<template>
  <v-card min-width="300px" max-width="400px" max-height="95vh" style="overflow: auto"
          class="elevation-0 card">
    <v-form v-model="valid">
      <v-container fluid @click.stop.prevent>
        <v-row>
          <v-col cols="12" class="d-flex pb-0">

            <v-spacer></v-spacer>
            <v-btn small outlined @click="close">Cancel</v-btn>
            <v-btn small color="primary" @click="save" :disabled="!valid">Save</v-btn>
          </v-col>
          <v-col cols="12">
            <v-text-field
              ref="column"
              hide-details
              color="primary"
              v-model="newColumn.cn"
              @input="newColumn.altered = newColumn.altered || 8"
              class="caption"
              label="Column name"
              dense outlined></v-text-field>

          </v-col>
          <div :class="{
          editDisabled :isEditDisabled
        }">
            <v-col cols="12" v-if="relation">
              <div class="caption">

                <p class="mb-1">Foreign Key </p>

                <v-icon small class="mt-n1">mdi-table</v-icon>
                <span class="text-capitalize font-weight-bold body-1"> {{ relation._rtn }}</span>
                <v-icon small
                        class="ml-3 mt-n1"
                        @click="deleteRelation('showDialog', column)"
                        color="error"
                        v-ge="['columns','fk-delete']"
                >mdi-delete-forever
                </v-icon>
                <span v-if="relation.type=== 'virtual'" class="caption">(v)</span>
              </div>
            </v-col>
            <template v-else>
              <v-col cols="12">
                <v-autocomplete
                  @change="onUiTypeChange"
                  v-model="newColumn.uidt"
                  hide-details
                  item-value="name"
                  item-text="name"
                  class="caption ui-type"
                  :class="{'primary lighten-5' : newColumn.uidt }"
                  label="Column type"
                  dense
                  outlined
                  :items="uiTypes">
                  <template v-slot:selection="{item}">
                    <div>
                      <v-icon color="grey darken-4" small class="mr-1">{{ item.icon }}</v-icon>
                      <span class="caption  grey--text text--darken-4"> {{ item.name }}</span>
                    </div>
                  </template>


                  <template v-slot:item="{item}">
                    <div class="caption">
                      <v-icon small class="mr-1">{{ item.icon }}</v-icon>
                      {{ item.name }}
                    </div>
                  </template>
                </v-autocomplete>


                <!--                        <v-list dense max-height="calc(100vh - 300px)" style="overflow: auto">-->
                <!--                          <v-list-item v-for="item in uiTypes" @click.stop :key="item">-->
                <!--                            <span class="caption">{{ item }}</span>-->
                <!--                          </v-list-item>-->
                <!--                        </v-list>-->
              </v-col>

              <template v-if="newColumn.uidt !== 'Formula'">
                <v-col cols="12"
                       v-if="isLinkToAnotherRecord">

                  <linked-to-another-options
                    ref="relation"
                    :column="newColumn"
                    :nodes="nodes"
                    @onColumnSelect="onRelColumnSelect"
                  ></linked-to-another-options>
                </v-col>
                <v-col cols="12"
                       v-if="isRelation">
                  <relation-options
                    ref="relation"
                    :column="newColumn"
                    :nodes="nodes"
                    @onColumnSelect="onRelColumnSelect"
                    :isSQLite="isSQLite"
                  ></relation-options>
                </v-col>

                <v-col cols="12" v-if="isSelect">
                  <custom-select-options
                    v-model="newColumn.dtxp"
                    @input="newColumn.altered = newColumn.altered || 2"
                  ></custom-select-options>
                </v-col>


                <template v-if="newColumn.cn && newColumn.uidt && !isLinkToAnotherRecord">
                  <v-col cols="12">
                    <v-container fluid class="wrapper">
                      <v-row>
                        <v-col cols="12">
                          <div class="d-flex justify-space-between caption">
                            <v-tooltip bottom z-index="99999">
                              <template v-slot:activator="{on}">
                                <div v-on="on">
                                  <v-checkbox
                                    :disabled="newColumn.pk || !sqlUi.columnEditable(newColumn)"
                                    class="mr-2 mt-0" dense hide-details label="NN"
                                    @input="newColumn.altered = newColumn.altered || 2"
                                    v-model="newColumn.rqd">
                                    <template v-slot:label>
                                      <span class="caption font-weight-bold">NN</span>
                                    </template>
                                  </v-checkbox>
                                </div>
                              </template>
                              <span>Not Null</span>
                            </v-tooltip>
                            <v-tooltip bottom z-index="99999">
                              <template v-slot:activator="{on}">
                                <div v-on="on">
                                  <v-checkbox

                                    v-model="newColumn.pk"
                                    :disabled="!sqlUi.columnEditable(newColumn)"
                                    @input="newColumn.altered = newColumn.altered || 2"
                                    class="mr-2 mt-0" dense hide-details label="PK">
                                    <template v-slot:label>
                                      <span class="caption font-weight-bold">PK</span>
                                    </template>
                                  </v-checkbox>
                                </div>
                              </template>
                              <span>Primary Key</span>
                            </v-tooltip>

                            <v-tooltip bottom z-index="99999">
                              <template v-slot:activator="{on}">
                                <div v-on="on">
                                  <v-checkbox

                                    @input="newColumn.altered = newColumn.altered || 2"
                                    :disabled="sqlUi.colPropUNDisabled(newColumn) || !sqlUi.columnEditable(newColumn)"
                                    class="mr-2 mt-0" dense hide-details label="AI"
                                    v-model="newColumn.ai">
                                    <template v-slot:label>
                                      <span class="caption font-weight-bold">AI</span>
                                    </template>
                                  </v-checkbox>
                                </div>
                              </template>
                              <span>Auto Increment</span>
                            </v-tooltip>


                            <v-tooltip bottom z-index="99999">
                              <template v-slot:activator="{on}">
                                <div v-on="on">
                                  <v-checkbox class="mr-2 mt-0" dense hide-details label="UN"
                                              @input="newColumn.altered = newColumn.altered || 2"
                                              :disabled="sqlUi.colPropUNDisabled(newColumn) || !sqlUi.columnEditable(newColumn)"
                                              v-model="newColumn.un">
                                    <template v-slot:label>
                                      <span class="caption font-weight-bold">UN</span>
                                    </template>
                                  </v-checkbox>
                                </div>
                              </template>
                              <span>Unsigned</span>
                            </v-tooltip>

                            <v-tooltip bottom z-index="99999">
                              <template v-slot:activator="{on}">
                                <div v-on="on">
                                  <v-checkbox class="mr-2 mt-0" dense hide-details label="UN"
                                              @input="newColumn.altered = newColumn.altered || 2"
                                              :disabled=" sqlUi.colPropAuDisabled(newColumn) || !sqlUi.columnEditable(newColumn)"
                                              v-model="newColumn.au">
                                    <template v-slot:label>
                                      <span class="caption font-weight-bold">AU</span>
                                    </template>
                                  </v-checkbox>
                                </div>
                              </template>
                              <span>Auto Update Timestamp</span>
                            </v-tooltip>
                          </div>
                        </v-col>
                        <v-col cols="12">
                          <v-autocomplete
                            v-model="newColumn.dt"
                            hide-details
                            class="caption data-type"
                            label="Type in Database"
                            dense
                            outlined
                            :items="dataTypes">
                          </v-autocomplete>
                        </v-col>

                        <v-col :cols="sqlUi.showScale(newColumn) && !isSelect ? 6 : 12">
                          <v-text-field
                            v-if="!isSelect"
                            dense
                            :disabled="sqlUi.getDefaultLengthIsDisabled(newColumn.dt) || !sqlUi.columnEditable(newColumn)"
                            class="caption"
                            label="Length / Values"
                            outlined
                            hide-details v-model="newColumn.dtxp"
                            @input="newColumn.altered = newColumn.altered || 2"
                          ></v-text-field>
                        </v-col>
                        <v-col :cols="isSelect ?12 : 6" v-if="sqlUi.showScale(newColumn)">

                          <v-text-field
                            dense
                            :disabled=" !sqlUi.columnEditable(newColumn)"
                            class="caption"
                            label="Scale"
                            outlined
                            @input="newColumn.altered = newColumn.altered || 2"
                            hide-details v-model="newColumn.dtxs"></v-text-field>
                        </v-col>

                        <v-col cols="12">
                          <v-textarea v-model="newColumn.cdf"
                                      @input="newColumn.altered = newColumn.altered || 2"
                                      label="Default value"
                                      :hint="sqlUi.getDefaultValueForDatatype(newColumn.dt)"
                                      persistent-hint
                                      rows="3"
                                      outlined dense class="caption"></v-textarea>
                        </v-col>
                      </v-row>
                    </v-container>
                  </v-col>
                </template>
              </template>
              <template v-else>
                <v-col cols12>
                  <v-autocomplete
                    label="Formula"
                    hide-details
                    class="caption formula-type"
                    outlined
                    dense
                    :items="formulas"
                  >

                    <template v-slot:item="{item}">
                      <span class="green--text text--darken-2 caption font-weight-regular">{{ item }}</span>
                    </template>

                  </v-autocomplete>
                </v-col>
              </template>
            </template>

            <div class="disabled-info" :class="{'d-none':!isEditDisabled}">
              <v-alert dense type="warning" icon="info" class="caption mx-2" outlined>
                This spreadsheet is connected to an SQLite DB.<br>
                For production please see <a href="https://github.com/nocodb/nocodb#production-setup"
                                             target="_blank">here</a>.
              </v-alert>
            </div>
          </div>


        </v-row>
      </v-container>
    </v-form>
    <dlg-label-submit-cancel
      type="primary"
      v-if="relationDeleteDlg"
      :dialogShow="relationDeleteDlg"
      :actionsMtd="deleteRelation"
      heading="Click Submit to Delete the Relation"
    />

  </v-card>
</template>

<script>
import {uiTypes} from "@/components/project/spreadsheet/helpers/uiTypes";
import CustomSelectOptions from "@/components/project/spreadsheet/editColumn/customSelectOptions";
import RelationOptions from "@/components/project/spreadsheet/editColumn/relationOptions";
import DlgLabelSubmitCancel from "@/components/utils/dlgLabelSubmitCancel";
import LinkedToAnotherOptions from "@/components/project/spreadsheet/editColumn/linkedToAnotherOptions";
import {SqliteUi} from "@/helpers/SqliteUi";

export default {
  name: "editColumn",
  components: {LinkedToAnotherOptions, DlgLabelSubmitCancel, RelationOptions, CustomSelectOptions},
  props: {
    nodes: Object,
    sqlUi: [Object, Function],
    meta: Object,
    editColumn: Boolean,
    column: Object,
    columnIndex: Number,
    value: Boolean
  },
  data: () => ({
    valid: false,
    relationDeleteDlg: false,
    newColumn: {},
    uiTypes,
    // dataTypes: [],
    formulas: ["AVERAGE()", "COUNT()", "COUNTA()", "COUNTALL()", "SUM()", "MIN()", "MAX()", "AND()", "OR()", "TRUE()", "FALSE()", "NOT()", "XOR()", "ISERROR()", "IF()", "LEN()", "MID()", "LEFT()", "RIGHT()", "FIND()", "CONCATENATE()", "T()", "VALUE()", "ARRAYJOIN()", "ARRAYUNIQUE()", "ARRAYCOMPACT()", "ARRAYFLATTEN()", "ROUND()", "ROUNDUP()", "ROUNDDOWN()", "INT()", "EVEN()", "ODD()", "MOD()", "LOG()", "EXP()", "POWER()", "SQRT()", "CEILING()", "FLOOR()", "ABS()", "RECORD_ID()", "CREATED_TIME()", "ERROR()", "BLANK()", "YEAR()", "MONTH()", "DAY()", "HOUR()", "MINUTE()", "SECOND()", "TODAY()", "NOW()", "WORKDAY()", "DATETIME_PARSE()", "DATETIME_FORMAT()", "SET_LOCALE()", "SET_TIMEZONE()", "DATESTR()", "TIMESTR()", "TONOW()", "FROMNOW()", "DATEADD()", "WEEKDAY()", "WEEKNUM()", "DATETIME_DIFF()", "WORKDAY_DIFF()", "IS_BEFORE()", "IS_SAME()", "IS_AFTER()", "REPLACE()", "REPT()", "LOWER()", "UPPER()", "TRIM()", "SUBSTITUTE()", "SEARCH()", "SWITCH()", "LAST_MODIFIED_TIME()", "ENCODE_URL_COMPONENT()", "REGEX_EXTRACT()", "REGEX_MATCH()", "REGEX_REPLACE()"]
  }),
  async created() {
    this.genColumnData();
    // await this.loadDataTypes();
  },
  methods: {
    onRelColumnSelect(colMeta) {
      Object.assign(this.newColumn, {
        dt: colMeta.dt,
        dtxp: colMeta.dtxp,
        dtxs: colMeta.dtxs,
        un: colMeta.un
      });
    },
    genColumnData() {
      this.newColumn = this.column ? {...this.column} : this.sqlUi.getNewColumn(this.meta.columns.length + 1);
      this.newColumn.cno = this.newColumn.cn;
    },
    /*
      async loadDataTypes() {
          try {
            const result = await this.$store.dispatch('sqlMgr/ActSqlOp', [{
              env: this.nodes.env,
              dbAlias: this.nodes.dbAlias
            }, 'getKnexDataTypes', {}])

            this.dataTypes = result.data.list;
          } catch (e) {
            this.$toast.error('Error loading datatypes :' + e).goAway(4000);
            throw e;
          }
        },
        */
    close() {
      this.$emit('close');
      this.newColumn = {};
    },
    async save() {
      try {

        if (this.newColumn.uidt === 'Formula') {
          return this.$toast.info('Coming Soon...').goAway(3000)
        }

        this.newColumn.tn = this.nodes.tn;
        this.newColumn._cn = this.newColumn.cn;

        const columns = [...this.meta.columns];

        if (columns.length) {
          columns[0].tn = this.nodes.tn;
        }

        if (this.editColumn) {
          columns[this.columnIndex] = this.newColumn;
        } else {
          columns.push(this.newColumn)
        }

        let result = await this.$store.dispatch('sqlMgr/ActSqlOpPlus', [{
          env: this.nodes.env,
          dbAlias: this.nodes.dbAlias
        }, "tableUpdate", {
          tn: this.nodes.tn,
          _tn: this.meta._tn,
          originalColumns: this.meta.columns,
          columns
        }]);


        if (this.isRelation && this.$refs.relation) {
          await this.$refs.relation.saveRelation();
        }


        this.$emit('saved');
      } catch (e) {
        console.log(e)
      }

      this.$emit('close');
    },
    onDataTypeChange() {

      this.newColumn.rqd = false;
      this.newColumn.pk = false;
      this.newColumn.ai = false;
      this.newColumn.cdf = null;
      this.newColumn.un = false;
      this.newColumn.dtxp = this.sqlUi.getDefaultLengthForDatatype(this.newColumn.dt);
      this.newColumn.dtxs = this.sqlUi.getDefaultScaleForDatatype(this.newColumn.dt);

      this.newColumn.dtx = 'specificType';

      this.$set(this.newColumn, 'uidt', this.sqlUi.getUIType(this.newColumn));

      this.newColumn.altered = this.newColumn.altered || 2;
    },
    onUiTypeChange() {
      const colProp = this.sqlUi.getDataTypeForUiType(this.newColumn);
      this.newColumn = {
        ...this.newColumn,
        rqd: false,
        pk: false,
        ai: false,
        cdf: null,
        un: false,
        dtx: 'specificType',
        ...colProp
      };

      this.newColumn.dtxp = this.sqlUi.getDefaultLengthForDatatype(this.newColumn.dt);
      this.newColumn.dtxs = this.sqlUi.getDefaultScaleForDatatype(this.newColumn.dt);

      this.newColumn.altered = this.newColumn.altered || 2;
    },
    focusInput() {
      setTimeout(() => {
        if (this.$refs.column && this.$refs.column.$el) {
          this.$refs.column.$el.querySelector('input').focus()
        }
      }, 100);
    },
    async deleteRelation(action = "", column) {

      try {
        if (action === "showDialog") {
          this.relationDeleteDlg = true;
        } else if (action === "hideDialog") {
          this.relationDeleteDlg = false;
        } else {
          let result = await this.$store.dispatch('sqlMgr/ActSqlOpPlus', [
            {
              env: this.nodes.env,
              dbAlias: this.nodes.dbAlias
            },
            this.relation.type === 'virtual' ? 'xcVirtualRelationDelete' : "relationDelete",
            {
              childColumn: this.relation.cn,
              childTable: this.nodes.tn,
              parentTable: this.relation
                .rtn,
              parentColumn: this.relation
                .rcn,
            }
          ]);
          console.log("relationDelete result ", result);
          // await this.loadColumnList();
          this.relationDeleteDlg = false;
          this.relation = null;
          this.$toast.success('Foreign Key deleted successfully').goAway(3000);
          this.$emit('onRelationDelete');
        }
      } catch (e) {
        console.log(e);
        this.$toast.error('Foreign key relation delete failed' + e).goAway(3000);
        throw e;
      }


    },
  }, watch: {
    column() {
      this.genColumnData();
    },
  }, mounted() {
    this.focusInput()
  },
  computed: {
    isEditDisabled() {
      return this.editColumn && this.isSQLite && !this.relation;
    },
    isSQLite() {
      return this.sqlUi === SqliteUi
    },
    dataTypes() {
      return this.sqlUi.getDataTypeListForUiType(this.newColumn)
    },
    isSelect() {
      return this.newColumn && (this.newColumn.uidt === 'MultiSelect'
        || this.newColumn.uidt === 'SingleSelect');
    },
    isRelation() {
      return this.newColumn && this.newColumn.uidt === 'ForeignKey';
    },
    isLinkToAnotherRecord() {
      return this.newColumn && this.newColumn.uidt === 'LinkToAnotherRecord';
    },
    relation() {
      return this.meta && this.column && this.meta.belongsTo && this.meta.belongsTo.find(bt => bt.cn === this.column.cn);
    }
  }

}
</script>

<style scoped lang="scss">

::v-deep {


  .v-input__slot {
    min-height: auto !important;
  }

  .v-input:not(.v-input--is-focused) fieldset {
    border-color: #7f828b33 !important;
  }

  .data-type, .ui-type, .formula-type {
    .v-input__append-inner {
      margin-top: 4px !important;
    }
  }

  .ui-type input {
    height: 24px;
  }

  .v-input--selection-controls__input > i {
    transform: scale(.83);
  }

  label {
    font-size: 0.75rem !important
  }

  .v-text-field--outlined.v-input--dense .v-label:not(.v-label--active) {
    top: 6px;
  }
}

.card {
  border: solid 2px #7f828b33;
}

.wrapper {
  border: solid 2px #7f828b33;
  border-radius: 4px;
}

.editDisabled {
  position: relative;

  .disabled-info {
    display: flex;
    align-items: center;
    justify-content: center;
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    bottom: 0;
    background: var(--v-backgroundColor-base);
    opacity: .9;

    & > * {
      opacity: 1;
    }
  }
}

</style>

<template>
  <v-container class="ma-0 pa-0"  fluid >
    <v-col cols="12" class="px-0">
      <v-toolbar text height="42" class="grey--text">
        {{ heading }}
        <v-spacer></v-spacer>
        <x-btn tooltip="Prettify SQL" small outlined @click="pretify" btn.class="grey--text">Prettify
        </x-btn>
      </v-toolbar
      >
      <monaco-editor
        :style="cssStyle ? cssStyle : ''"
        class="editor card"
        ref="editor"
        @selection="selectionFn"
        theme="vs-dark"
        v-model="codeLocal"
        language="sql"
        :minimap="minimap"
        :readOnly="readOnly"
      >
      </monaco-editor
      >

    </v-col>

  </v-container>
</template>

<script>
  import MonacoEditor from "./index.js";
  import sqlFormatter from "sql-formatter";

  export default {
    ssr: false,
    components: {
      MonacoEditor,
      // sqlFormatter
    },
    beforeCreate() {
      // console.log(MonacoEditor)
    },
    data() {
      return {
        codeLocal: `${this.code || ""}`,
        selection: null,
        minimap: {
          enabled: true
        }
      };
    },
    computed: {},
    props: ["code", "cssStyle", "readOnly", "heading"],
    methods: {
      selectionFn() {
        const editor = this.$refs.editor.getMonaco();
        const range = editor.getSelection();
        const selectedText = editor.getModel().getValueInRange(range);
        // console.log('getValue', editor.getModel())
        this.selection = selectedText;
        this.selectionRange = range;
      },
      pretify() {
        // console.log("this.code", this.code);
        const editor = this.$refs.editor.getMonaco();

        if (this.selection && this.selectionRange) {
          const op = {
            identifier: "prettifySelection",
            range: this.selectionRange,
            text: sqlFormatter.format(this.selection),
            forceMoveMarkers: true
          };
          editor.executeEdits("sqlFormatter", [op]);
          this.selection = null;
          this.selectionRange = null;
        } else {
          // console.log("selected format before:: ", this.codeLocal);
          const op = {
            identifier: "prettifyDoc",
            range: editor.getModel().getFullModelRange(),
            text: sqlFormatter.format(this.codeLocal || ""),
            forceMoveMarkers: true
          };
          editor.executeEdits("sqlFormatter", [op]);
        }
      },
      toggleMiniMap() {
        const editor = this.$refs.editor.getMonaco();
        this.minimap.enabled = !this.minimap.enabled;
        editor.updateOptions({
          minimap: {
            enabled: this.minimap.enabled
          }
        });
      }
    },
    watch: {
      codeLocal: function (newValue) {
        //INFO: for updating value of prop `code` in parent comp
        // console.log("update:code Event Emitted", newValue);
        this.$emit("update:code", newValue);
      },
      code: function (newValue) {
        this.codeLocal = newValue;
      }
    },
    created() {
      //
    }
  };
</script>

<style>
  .editor {
    /* width: 100%;
    height: 800px; */
  }
</style>
<!--
/**
 * @copyright Copyright (c) 2021, Xgene Cloud Ltd
 *
 * @author Naveen MR <oof1lab@gmail.com>
 * @author Pranav C Balan <pranavxc@gmail.com>
 *
 * @license GNU AGPL version 3 or any later version
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU Affero General Public License as
 * published by the Free Software Foundation, either version 3 of the
 * License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU Affero General Public License for more details.
 *
 * You should have received a copy of the GNU Affero General Public License
 * along with this program. If not, see <http://www.gnu.org/licenses/>.
 *
 */
-->

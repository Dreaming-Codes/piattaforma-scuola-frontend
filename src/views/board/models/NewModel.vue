<template>
  <v-container class="fill-height pa-0" fluid>
    <v-card elevation="12" width="100%">
      <div class="mx-3 mt-4">
        <v-text-field v-model="name" class="" label="Nome"></v-text-field>
        <v-textarea v-model="description" height="5%" label="Descrizione"></v-textarea>
        <input ref="fileHandler" accept="application/pdf" class="d-none" type="file" @change="pdfLoader">
        <v-card v-if="!file" class="mx-auto mt-3" color="other" elevation="6" min-height="200" @dragenter.prevent
                @dragover.prevent @drop.prevent="pdfLoader">
          <v-card-actions class="justify-center fill-height">
            <v-icon class="mt-8" size="120" @click="openFileHandler">mdi-cloud-upload</v-icon>
          </v-card-actions>
        </v-card>
        <editor v-else ref="editor" :pdf="file" style="width: 100%"></editor>
      </div>
      <v-card-actions>
        <v-spacer/>
        <v-btn :disabled="loading" color="error" text @click="$router.push('/board/models')">Annulla</v-btn>
        <v-btn :loading="loading" color="accent" text @click="saveModel">Salva</v-btn>
      </v-card-actions>
    </v-card>

  </v-container>
</template>

<script lang="ts">
import Editor from "@/components/PDP/Editor.vue";
import Component from "vue-class-component";
import Vue from "vue";
import gql from "graphql-tag";

@Component({
  components: {
    Editor
  }
})
export default class NewModel extends Vue {
  file = null;
  name = "";
  description = "";

  loading = false;

  $refs!: {
    fileHandler: HTMLInputElement,
    editor: Editor
  }

  openFileHandler() {
    this.$refs.fileHandler.click()
  }

  pdfLoader(event) {
    this.file = event.target.files[0];
  }

  async saveModel() {
    this.loading = true;
    console.log(this.$refs.editor.getFields())
    await this.$apollo.mutate({
      mutation: gql`mutation newDisorder($modelDocument: InputDisorder!, $pdfFile: Upload!){newDisorder(disorderDocument: $modelDocument, pdf: $pdfFile)}`,
      variables: {
        modelDocument: {
          name: this.name,
          description: this.description,
          fields: this.$refs.editor.getFields()
        },
        pdfFile: new Blob([await this.$refs.editor.getPDF()], {
          type: "application/pdf"
        }),
      },
    })

    this.$router.push('/board/models')
  }
}
</script>

<style scoped>

</style>

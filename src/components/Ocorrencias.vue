<template>
  <v-card>
    <v-card-title>
      <v-text-field
        v-model="search"
        append-icon="mdi-magnify"
        label="Buscar nos registros"
        single-line
        hide-details
      ></v-text-field>
    </v-card-title>
    <v-data-table
      :headers="headers"
      :search="search"
      :items="ocorrencias"
      :no-data-text="'Nenhum registro'"
      :footer-props="optionsTableFooter"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Ocorrências de Perda</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                Adicionar registro
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="text-h5">Formulário de Perda</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        :rules="ruleRequired"
                        v-model="editedItem.nome"
                        label="Nome"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        :rules="cpfRules"
                        v-model="editedItem.cpf"
                        label="CPF"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12" sm="8" md="8">
                      <v-text-field
                        type="email"
                        :rules="emailRules"
                        v-model="editedItem.email"
                        label="E-mail"
                      ></v-text-field>
                    </v-col>
                     <v-col cols="12" sm="4" md="4">
                      <v-menu
                        v-model="menu2"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        transition="scale-transition"
                        offset-y
                        min-width="auto"
                      >
                        <template v-slot:activator="{ on, attrs }">
                          <v-text-field
                            v-model="editedItem.data_colheita"
                            label="Data da colheita"
                            prepend-icon="mdi-calendar"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-date-picker
                          :rules="ruleRequired"
                          v-model="editedItem.data_colheita"
                          @input="menu2 = false"
                        ></v-date-picker>
                      </v-menu>
                    </v-col>
                  </v-row>
                  <v-row>
                   
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field
                        :rules="ruleRequired"
                        v-model="editedItem.lavoura"
                        label="Lavoura"
                      ></v-text-field>
                    </v-col>
                    <v-col>
                      <v-select
                        v-model="editedItem.causa"
                        :items="causas"
                        label="Causa"
                      ></v-select>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        :rules="coordRules"
                        v-model="editedItem.lat"
                        label="Latitude"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        :rules="coordRules"
                        v-model="editedItem.lon"
                        label="Longitude"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <strong>{{ validaOcorrencia }}</strong>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">
                  Cancelar
                </v-btn>
                <v-btn color="blue darken-1" text @click="save"> Salvar </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="text-h5"
                >Tem certeza que deseja deletar este item?</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete"
                  >Cancelar</v-btn
                >
                <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                  >OK</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:[`item.actions`]="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
        <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
import axios from "axios";
import { mask } from "vue-the-mask";
export default {
  name: "Ocorrencias",
  props: {
    msg: String,
  },
  directives: { mask },
  data() {
    return {
      optionsTableFooter: {
        itemsPerPageText: 'Ocorrências por página:'
      },
      causas: [
        "Chuva excessiva",
        "Geada",
        "Granizo",
        "Seca",
        "Vendaval",
        "Raio",
      ],
      disabled: false,
      readonly: false,
      chips: false,
      multiple: false,
      appendIcon: false,
      appendSlot: false,
      appendItemSlot: false,
      prependIcon: false,
      prependSlot: false,
      prependItemSlot: false,
      selectSlot: false,
      model: "Foo",
      search: "",
      valid: true,
      dialog: false,
      dialogDelete: false,
      date: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
        .toISOString()
        .substr(0, 10),
      menu: false,
      modal: false,
      menu2: false,
      headers: [
        { text: "Nome", value: "nome" },
        { text: "E-mail", value: "email" },
        { text: "CPF", value: "cpf" },
        { text: "Data Colheita", value: "data_colheita" },
        { text: "Lavoura", value: "lavoura" },
        { text: "Causa", value: "causa" },
        { text: "Latitude", value: "lat" },
        { text: "Longitude", value: "lon" },
        { text: "Opções", value: "actions", sortable: false },
      ],
      ruleRequired: [(v) => !!v || "Campo necessário"],
      coordRules: [(v) => this.validarCoord(v) || "Coordenada inválida"],
      cpfRules: [
        (v) => !!v || "Digite o número do CPF",
        (v) => this.validarCPF(v) || "CPF inválido",
      ],
      emailRules: [
        (v) => !!v || "Digite um E-mail",
        (v) => /.+@.+\..+/.test(v) || "Digite um E-mail válido",
      ],
      ocorrencias: [],
      editedIndex: -1,
      editedItem: {
        url: "",
        nome: "",
        email: "",
        cpf: "",
        data_colheita: "",
        lavoura: "",
        causa: "",
        lat: "",
        lon: "",
      },
      defaultItem: {
        url: "",
        nome: "",
        email: "",
        cpf: "",
        data_colheita: "",
        lavoura: "",
        causa: "",
        lat: "",
        lon: "",
      },
    };
  },
  watch: {
    dialog(val) {
      val || this.close();
    },
    dialogDelete(val) {
      val || this.closeDelete();
    },
  },
  mounted() {
    this.getAll();
  },
  methods: {
    getAll() {
      axios.get("http://127.0.0.1:8000/ocorrencias").then((res) => {
        this.ocorrencias = res.data;
      });
    },
    getOne(ocorrencia) {
      this.url = ocorrencia.url;
      this.nome = ocorrencia.nome;
      this.email = ocorrencia.email;
      this.cpf = ocorrencia.cpf;
      this.data_colheita = ocorrencia.data_colheita;
      this.lavoura = ocorrencia.lavoura;
      this.causa = ocorrencia.causa;
      this.lat = ocorrencia.lat;
      this.lon = ocorrencia.lon;
    },
    deleteOne(url) {
      axios.delete(url, {
        auth: {
          username: "augustoadmin",
          password: "987654321",
        },
      });
    },
    postOccor(form) {
      axios.post(
        "http://127.0.0.1:8000/ocorrencias/",
        {
          nome: form.nome,
          email: form.email,
          cpf: form.cpf,
          data_colheita: form.data_colheita,
          lavoura: form.lavoura,
          causa: form.causa,
          lat: form.lat,
          lon: form.lon,
        },
        {
          auth: {
            username: "augustoadmin",
            password: "987654321",
          },
        }
      );
    },
    atualizOccor(form) {
      axios.put(
        form.url,
        {
          nome: form.nome,
          email: form.email,
          cpf: form.cpf,
          data_colheita: form.data_colheita,
          lavoura: form.lavoura,
          causa: form.causa,
          lat: form.lat,
          lon: form.lon,
        },
        {
          auth: {
            username: "augustoadmin",
            password: "987654321",
          },
        }
      );
    },
    validate() {
      this.$refs.form.validate();
    },
    editItem(item) {
      this.editedIndex = this.ocorrencias.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },

    deleteItem(item) {
      this.editedIndex = this.ocorrencias.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    deleteItemConfirm() {
      this.ocorrencias.splice(this.editedIndex, 1);
      this.closeDelete();
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      //eslint-disable-next-line
      debugger;
      if (this.editedIndex > -1) {
        Object.assign(this.ocorrencias[this.editedIndex], this.editedItem);
        this.atualizOccor(this.editedItem);
      } else {
        this.ocorrencias.push(this.editedItem);
        this.postOccor(this.editedItem);
      }
      this.close();
    },
    validarCPF(inputCPF) {
      if (!inputCPF) {
        return;
      }
      var soma = 0;
      var i;
      var resto;
      inputCPF = inputCPF.trim();
      inputCPF = inputCPF.replace(/\./g, "");
      inputCPF = inputCPF.replace("-", "");

      if (inputCPF == "00000000000") return false;
      for (i = 1; i <= 9; i++)
        soma = soma + parseInt(inputCPF.substring(i - 1, i)) * (11 - i);
      resto = (soma * 10) % 11;

      if (resto == 10 || resto == 11) resto = 0;
      if (resto != parseInt(inputCPF.substring(9, 10))) return false;

      soma = 0;
      for (i = 1; i <= 10; i++)
        soma = soma + parseInt(inputCPF.substring(i - 1, i)) * (12 - i);
      resto = (soma * 10) % 11;

      if (resto == 10 || resto == 11) resto = 0;
      if (resto != parseInt(inputCPF.substring(10, 11))) return false;
      return true;
    },
    validarCoord(coord) {
      let pattern = new RegExp("^-?([1-8]?[1-9]|[1-9]0)\\.{1}\\d{1,6}");

      return pattern.test(coord);
    },
    calculaDistancia(lat1, lon1, lat2, lon2) {
      var R = 6371; // km
      var dLat = this.toRad(lat2 - lat1);
      var dLon = this.toRad(lon2 - lon1);
      lat1 = this.toRad(lat1);
      lat2 = this.toRad(lat2);

      var a =
        Math.sin(dLat / 2) * Math.sin(dLat / 2) +
        Math.sin(dLon / 2) *
          Math.sin(dLon / 2) *
          Math.cos(lat1) *
          Math.cos(lat2);
      var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
      var d = R * c;
      console.log(d);
      return d;
    },

    toRad(Value) {
      return (Value * Math.PI) / 180;
    },
  },

  computed: {
    validaOcorrencia() {
      let mensagem = "";
      this.ocorrencias.forEach((ocorrencia) => {
        var distancia = this.calculaDistancia(
          ocorrencia.lat,
          ocorrencia.lon,
          this.editedItem.lat,
          this.editedItem.lon
        );
        if (
          distancia < 10 &&
          ocorrencia.data_colheita === this.editedItem.data_colheita &&
          ocorrencia.causa != this.editedItem.causa
        ) {
          mensagem = "Evento divergente!!!";
        }
      });
      return mensagem;
    },
    formTitle() {
      return this.editedIndex === -1 ? "Novo item" : "Editar Item";
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>

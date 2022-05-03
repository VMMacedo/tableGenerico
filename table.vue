<template>
  <div>
    <Loader v-if="this.$store.state.loading" />
    <Alert
      v-if="this.$store.state.showAlert"
      :typeAlert="this.$store.state.typeAlert"
      :msgAlert="this.$store.state.msgAlert"
    />
    <div v-if="!this.$store.state.loading && this.$store.state.loadingTable">
      <div class="row">
        <div class="col-sm-12 col-md-5">
          <div
            class="btn-group-sm"
            role="group"
            aria-label="Basic mixed styles example"
          >
            <button
              type="button"
              class="btn btn-primary me-1 text-white"
              @click="setButton('', 'store', 'Adicionar')"
              :data-bs-toggle="databstoggle"
              :data-bs-target="databstarget"
            >
              Adicionar
            </button>
            <button
              type="button"
              class="btn btn-outline-primary me-1"
              style="color: red"
              @click="generateExport('pdf')"
            >
              PDF
            </button>
            <button
              type="button"
              class="btn btn-outline-primary me-1"
              style="color: green"
              @click="generateExport('csv')"
            >
              CSV
            </button>
          </div>
        </div>

        <div class="col-sm-12 col-md-4 offset-md-3">
          <div class="input-group input-group-sm mb-3">
            <input
              type="text"
              class="form-control"
              aria-describedby="button-addon2"
              v-model="search"
              @keyup.enter="btnSearch()"
            />
            <button
              class="btn btn-outline-secondary"
              type="button"
              id="button-addon2"
              @click="btnSearch()"
            >
              Pesquisar
            </button>
          </div>
        </div>
      </div>
      <div class="table-responsive">
        <table class="table table-sm">
          <thead>
            <tr>
              <th
                v-for="(h, keyHeader) in titulos"
                :key="keyHeader"
                @click="ordenaCollum(h.var)"
              >
                {{ h.titulo }}
                <span class="material-icons icons-thead"> swap_vert </span>
              </th>
              <th class="text-center" :colspan="theadColSpan">Opções</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(obj, chave) in dadosFiltrados" :key="chave">
              <td
                class="align-middle"
                v-for="(valor, chaveValor) in obj"
                :key="chaveValor"
              >
                <template v-for="(t, chaveT) in titulos">
                  <template v-if="t['var'] === chaveValor">
                    <span v-if="t['tipo'] === 'text'" :key="chaveT">
                      {{ tratarText(valor) }}
                    </span>
                    <span v-if="t['tipo'] === 'date'" :key="chaveT">
                      {{ tratarDate(valor) }}
                    </span>
                    <span v-if="t['tipo'] === 'image'" :key="chaveT">
                      {{ tratarImage(valor) }}
                    </span>
                    <span v-if="t['tipo'] === 'float'" :key="chaveT">
                      {{ tratarFloat(valor) }}
                    </span>
                    <span v-if="t['tipo'] === 'monetario'" :key="chaveT">
                      {{ tratarMonetario(valor) }}
                    </span>
                    <span v-if="t['tipo'] === 'badge'" :key="chaveT">
                      <span v-html="tratarBagde(t, valor)"></span>
                    </span>
                  </template>
                </template>
              </td>
              <td
                class="text-center"
                v-for="(btn, keyBtn) in buttons"
                :key="keyBtn"
              >
                <jet-button
                  :type="type"
                  :classButton="btn.class"
                  :classDivBtn="classDivBtn"
                  :disabled="btn.disabled"
                  :databstoggle="databstoggle"
                  :databstarget="databstarget"
                  @click="setButton(obj, btn.action, btn.tituloModal)"
                >
                  <icon-span
                    :iconeTipo="btn.iconeTipo"
                    classMaterial="material-icons"
                  ></icon-span>
                </jet-button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="row justify-content-end">
        <div class="col-6 d-flex justify-content-start">
          <ul class="list-inline">
            <ul class="list-inline">
              <li class="list-inline-item">Mostrar</li>
              <li class="list-inline-item">
                <select
                  v-model="perPage"
                  class="form-select form-select-sm"
                  aria-label=".form-select-sm example"
                >
                  <option selected value="50">50</option>
                  <option value="100">100</option>
                  <option value="200">200</option>
                  <option value="300">300</option>
                </select>
              </li>
              <li class="list-inline-item">de {{ this.dadosTotal }}</li>
            </ul>
          </ul>
        </div>
        <div class="col-6 d-flex justify-content-end">
          <paginate>
            <li
              v-for="(paginate, key) in this.paginateLinks"
              :key="key"
              :class="paginate.active ? 'page-item active' : 'page-item'"
              @click="paginacao(paginate)"
            >
              <a class="page-link" href="#" v-html="paginate.label"></a>
            </li>
          </paginate>
        </div>
      </div>
    </div>
    <!--Modal-->
    <modal-component
      :modalID="modalID"
      :modalBtnSubmit="modalBtnSubmit"
      :colBtnModal="colBtnModal"
      :modalBtnTitleSalvar="modalBtnTitleSalvar"
      :modalDadosInp="modalDadosInp"
      :modalButtonId="modalButtonId"
      :modalTitle="modalTitle"
      ref="modal.btn"
    >
      <template v-slot:modalBody>
        <loader v-if="loadModal"></loader>
        <div v-if="layoutModal == 'show'">
          <table-layout>
            <tbody>
              <template v-for="md in dadosFiltradosModal">
                <tr v-for="(dados, dadosKey) in md" :key="dadosKey">
                  <th class="align-middle">{{ dadosKey }}</th>
                  <td class="align-middle" v-if="dados.inputype == 'text'">
                    {{ dados.valor }}
                  </td>
                  <td class="align-middle" v-if="dados.inputype == 'date'">
                    {{ tratarDate(dados.valor) }}
                  </td>
                  <template v-if="dados.inputype == 'array'">
                    <td class="align-middle">
                      <ul class="list-unstyled my-auto">
                        <li
                          v-for="(d, keyDados) in dados.valor"
                          :key="keyDados"
                        >
                          {{ d }}
                        </li>
                      </ul>
                    </td>
                  </template>
                </tr>
              </template>
            </tbody>
          </table-layout>
        </div>

        <form
          action=""
          id="formSubmitTable"
          autocomplete="off"
          @submit.prevent="formSubmit($event, dataId)"
        >
          <div v-if="layoutModal == 'destroy'">
            <h5>
              Deseja excluir <b>{{ modalNameItem }}?</b>
            </h5>
          </div>

          <div v-if="layoutModal == 'store'">
            <span v-for="(inputs, inputsKey) in modalDadosInp" :key="inputsKey">
              <span
                v-if="
                  inputs.inputType == 'text' || inputs.inputType == 'password'
                "
              >
                <jet-input
                  :id="inputs.inputVar"
                  :texto="inputs.inputTitulo"
                  :type="inputs.inputType"
                  :label="inputs.inputTitulo"
                  :required="inputs.inputRequired"
                  autofocus="false"
                  autocomplete="off"
                  :name="inputs.inputVar"
                >
                </jet-input>
              </span>

              <span v-else-if="inputs.inputType == 'select'">
                <span-select
                  :optionsSelectRoute="inputs.optionsSelectRoute"
                  :optionsSelectMethod="inputs.optionsSelectMethod"
                  :optionsSelectData="inputs.optionsSelectData"
                  :optionsSelectId="inputs.optionsSelectId"
                  :optionsSelectVar1="inputs.optionsSelectVar1"
                  :inputVar="inputs.inputVar"
                >
                </span-select>
              </span>

              <span v-else-if="inputs.inputType == 'checkbox'">
                <span-checkbox
                  :optionsSelectRoute="inputs.optionsSelectRoute"
                  :optionsSelectMethod="inputs.optionsSelectMethod"
                  :optionsSelectData="inputs.optionsSelectData"
                  :optionsSelectId="inputs.optionsSelectId"
                  :optionsSelectVar1="inputs.optionsSelectVar1"
                  :inputTitulo="inputs.inputTitulo"
                  :optionsSelectVar2="inputs.optionsSelectVar2"
                  :inputData="inputs.inputData"
                  :dadosFiltradosModal="dadosFiltradosModal"
                >
                </span-checkbox>
              </span>
            </span>
          </div>
          <div v-if="layoutModal == 'update'">
            <span v-for="(inputs, inputsKey) in modalDadosInp" :key="inputsKey">
              <span
                v-if="
                  inputs.inputType == 'text' || inputs.inputType == 'password'
                "
              >
                <jet-input
                  :id="inputs.inputVar"
                  :texto="inputs.inputTitulo"
                  :type="inputs.inputType"
                  :label="inputs.inputTitulo"
                  :required="inputs.inputRequired"
                  :modelValue="dadosInputUpdateSelect(inputs.inputVar)"
                  autofocus="false"
                  autocomplete="off"
                  :name="inputs.inputVar"
                >
                </jet-input>
              </span>
              <span v-else-if="inputs.inputType == 'select'">
                <span-select
                  :optionsSelectRoute="inputs.optionsSelectRoute"
                  :optionsSelectMethod="inputs.optionsSelectMethod"
                  :optionsSelectData="inputs.optionsSelectData"
                  :optionsSelectId="inputs.optionsSelectId"
                  :optionsSelectVar1="inputs.optionsSelectVar1"
                  :inputVar="inputs.inputVar"
                >
                </span-select>
              </span>

              <span v-else-if="inputs.inputType == 'checkbox'">
                <span-checkbox
                  :optionsSelectRoute="inputs.optionsSelectRoute"
                  :optionsSelectMethod="inputs.optionsSelectMethod"
                  :optionsSelectData="inputs.optionsSelectData"
                  :optionsSelectId="inputs.optionsSelectId"
                  :optionsSelectVar1="inputs.optionsSelectVar1"
                  :inputTitulo="inputs.inputTitulo"
                  :optionsSelectVar2="inputs.optionsSelectVar2"
                  :inputData="inputs.inputData"
                  :dadosFiltradosModal="dadosFiltradosModal"
                >
                </span-checkbox>
              </span>
            </span>
          </div>
        </form>
      </template>
      <template v-slot:modalFooter>
        <div class="row">
          <div :class="colBtnModal">
            <div class="d-grid gap-2 col-12 mx-auto">
              <button
                type="button"
                class="btn btn-secondary"
                data-bs-dismiss="modal"
                id="btnCloseModal"
              >
                Fechar
              </button>
            </div>
          </div>
          <div class="col-6" v-if="modalBtnSalvar">
            <div class="d-grid gap-2 col-12 mx-auto">
              <button
                :type="typeButtonFooter"
                :class="modalBtnSubmit"
                :id="modalButtonId"
                :disabled="$store.state.loadingButton"
                :form="formUpdateId"
              >
                <div
                  v-show="$store.state.loadingButton"
                  class="spinner-border spinner-border-sm"
                  role="status"
                >
                  <span class="visually-hidden">Loading...</span>
                </div>
                {{ modalBtnTitleSalvar }}
              </button>
            </div>
          </div>
        </div>
      </template>
    </modal-component>
  </div>
</template>
  
<script>
import Loader from "@/components/Loader.vue";
import Alert from "@/components/Alerts.vue";
import Paginate from "@/components/Paginate.vue";
import JetButton from "@/components/Button.vue";
import JetInput from "@/components/Input.vue";
import JetSelect from "@/components/Select.vue";
import IconSpan from "@/components/IconSpan.vue";
import ModalComponent from "@/components/Modal.vue";
import TableLayout from "@/components/TableLayout.vue";
import Checkbox from "@/components/Checkbox.vue";

import { defineAsyncComponent } from "vue";
export default {
  props: [
    "titulos",
    "buttons",
    "primaryKey",
    "modalSlot",
    "secondKey",
    "headerTituloValor",
  ],
  components: {
    Loader,
    Alert,
    Paginate,
    JetButton,
    IconSpan,
    ModalComponent,
    TableLayout,
    JetInput,
    JetSelect,
    Checkbox,
    SpanSelect: defineAsyncComponent({
      // the loader function
      loader: () => import("@/components/SpanSelect.vue"),

      // A component to use while the async component is loading
      loadingComponent: Loader,
      // Delay before showing the loading component. Default: 200ms.
      delay: 200,
    }),
    SpanCheckbox: defineAsyncComponent({
      // the loader function
      loader: () => import("@/components/SpanCheckbox.vue"),

      // A component to use while the async component is loading
      loadingComponent: Loader,
      // Delay before showing the loading component. Default: 200ms.
      delay: 100,
    }),
  },
  data() {
    return {
      retorno: "",
      perPage: 50,
      dados: "",
      search: "",
      data: "",
      dadosTotal: 0,
      paginateLinks: "",
      paginate: false,
      ordenaVal: "asc",
      type: "button",
      classDivBtn: "",
      rota: "",
      databstoggle: "modal",
      databstarget: "#modalTable",
      modalID: "modalTable",
      modalBtnSubmit: "btn btn-primary",
      modalBtnSalvar: true,
      colBtnModal: "col-6",
      modalBtnTitleSalvar: "Salvar",
      dadosModal: "",
      modalBody: {},
      modalDadosFiltrados: "",
      modalDadosInp: {},
      loadModal: true,
      modalButtonId: "",
      layoutModal: "",
      modalTitle: "Modal",
      dataId: "",
      dataAction: "",
      modalNameItem: "",
      selectModel: "",
      selectDados: "",
      modalIdUpdate: "",
      typeButtonFooter: {
        default: "submit",
      },
      formUpdateId: {
        default: "",
      },
      dadosUpdate: "",
    };
  },
  computed: {
    dadosFiltradosModal() {
      let dadosFilter = [];
      let newVal = {};

      if (Object.entries(this.modalBody).length > 0) {
        for (let [key, value] of Object.entries(this.modalDadosInp)) {
          for (let [key2, value2] of Object.entries(this.modalBody)) {
            let arrayKey = value.inputVar.split(".");
            let campo = value.inputTitulo;
            if (arrayKey.length > 1) {
              if (arrayKey[0] == key2) {
                let ObjectBody = this.modalBody;
                let cont = 0;

                for (let i = 0; i < arrayKey.length; i++) {
                  let vartemp = [];

                  if (Object.keys(ObjectBody).length > 0 && cont > 0) {
                    for (let [keyArr, valArr] of Object.entries(ObjectBody)) {
                      if (valArr.hasOwnProperty(arrayKey[i])) {
                        if (
                          typeof valArr[arrayKey[i]] === "object" &&
                          Object.keys(valArr[arrayKey[i]]).length > 0
                        ) {
                          i++;
                          for (let [keyArr2, valArr2] of Object.entries(
                            valArr[arrayKey[i - 1]]
                          )) {
                            if (arrayKey[i] === keyArr2) {
                              vartemp.push(valArr2);
                            }
                          }
                          i--;
                          ObjectBody = vartemp;
                        } else {
                          ObjectBody = valArr[arrayKey[i]];
                        }
                        cont = -1;
                      }
                    }
                  } else if (ObjectBody !== null) {
                    /**Função recursiva */
                    for (let [key3, value3] of Object.entries(ObjectBody)) {
                      if (arrayKey[i] == key3) {
                        ObjectBody = value3;
                      }
                    }
                  } else {
                    ObjectBody = "";
                  }
                  cont++;
                }
                newVal[campo] = {
                  valor: ObjectBody,
                  inputype: value.inputType,
                };
              }
            } else {
              if (value.inputVar == key2) {
                newVal[campo] = {
                  valor: value2,
                  inputype: value.inputType,
                };
              }
            }
          }
        }
        dadosFilter.push(newVal);
        return dadosFilter;
      }
    },

    theadColSpan() {
      return Object.keys(this.buttons).length;
    },
    dadosFiltrados() {
      let dadosFiltrados = [];
      if (this.dados != "") {
        let campos = [];

        //Tratamento dos Array dos campos
        for (let [key, value] of Object.entries(this.titulos)) {
          let vars = value.var.split(".");
          if (vars.length > 1) {
            let arrayVars = [];
            for (let i = 1; i < vars.length; i++) {
              arrayVars.push(vars[i]);
            }
            let variable = new Array();
            variable[vars[0]] = arrayVars;
            campos.push(variable);
          } else {
            campos.push(vars[0]);
          }
        }

        for (let [key, value] of Object.entries(this.dados.success)) {
          this.dadosTotal = value.total !== null ? value.total : 0;
          this.paginateLinks = value.links !== null ? value.links : null;

          if (value.data) {
            value.data.map((item, chave) => {
              let itemFiltrado = {};
              campos.forEach((campo) => {
                if (Array.isArray(campo)) {
                  let campoTratado = "";
                  let campoTratado2 = [];
                  let cont = 0;
                  for (var k in campo) {
                    /**Pegar o 1° array */
                    if (cont === 0) {
                      campoTratado = k;
                    }
                    campoTratado2.push(campo[k][0]);
                    /**TRATAMENTO DE NÍVEIS */
                    if (typeof campo[k][1] !== "undefined") {
                      campoTratado2.push(campo[k][1]);
                    }
                    if (typeof campo[k][2] !== "undefined") {
                      campoTratado2.push(campo[k][2]);
                    }
                    if (typeof campo[k][3] !== "undefined") {
                      campoTratado2.push(campo[k][3]);
                    }
                    cont++;
                  }
                  let varCamp = "";
                  campoTratado2.forEach((varc) => {
                    varCamp += "." + varc;
                  });
                  k = k + varCamp;

                  if (
                    item[campoTratado] !== null &&
                    item[campoTratado].length > 0
                  ) {
                    /**Verificar a quantidade  do array dimensional - IMPORTANTE */
                    if (campoTratado2.length === 1) {
                      itemFiltrado[k] = item[campoTratado][0][campoTratado2];
                    } else if (campoTratado2.length === 2) {
                      let c0 = campoTratado2[0];
                      let c1 = campoTratado2[1];
                      itemFiltrado[k] = item[campoTratado][0][c0][c1];
                    } else if (campoTratado2.length === 3) {
                      let c0 = campoTratado2[0];
                      let c1 = campoTratado2[1];
                      let c2 = campoTratado2[2];
                      itemFiltrado[k] = item[campoTratado][c0][c1][c2];
                    } else if (campoTratado2.length === 4) {
                      let c0 = campoTratado2[0];
                      let c1 = campoTratado2[1];
                      let c2 = campoTratado2[2];
                      let c3 = campoTratado2[3];
                      itemFiltrado[k] = item[campoTratado][c0][c1][c2][c3];
                    }
                  } else {
                    itemFiltrado[k] = "";
                  }
                } else {
                  itemFiltrado[campo] = item[campo];
                }
              });
              dadosFiltrados.push(itemFiltrado);
            });
            return dadosFiltrados;
          }
        }
      }
    },
  },
  methods: {
    async generateExport(type) {
      var dados,
        titulos,
        header = "";
      var rota = "exportApi";
      var method = "POST";
      var data = new Object();
      if (this.dados.success) {
        for (let [key, value] of Object.entries(this.dados.success)) {
          dados = value;
        }
        titulos = this.titulos;
        header = this.headerTituloValor;
        data.dados = dados;
        data.titulos = titulos;
        data.type = type;
        data.header = header;
        await this.genericApi(rota, method, data);
        if (type == "pdf") {
          const linkSource = `data:application/pdf;base64,${this.selectDados.dataBase64}`;
          const downloadLink = document.createElement("a");
          const fileName = "" + header + ".pdf";
          downloadLink.href = linkSource;
          downloadLink.download = fileName;
          downloadLink.click();
        } else if (type == "csv") {
          const atobCsv = decodeURIComponent(
            escape(atob(this.selectDados.dataBase64))
          );
          const linkSource = "data:text/csv;charset=utf-8," + atobCsv;
          const downloadLink = document.createElement("a");
          const fileName = "" + header + ".csv";
          downloadLink.href = linkSource;
          downloadLink.download = fileName;
          downloadLink.click();
        }
      } else {
        this.$toast.open({
          message: "Não foi possível ler arquivo PDF",
          type: "danger",
          dismissible: true,
          position: "bottom-right",
        });
      }
    },
    formSubmit(e, id) {
      switch (this.dataAction) {
        case "store":
          var objeto = {};
          var checkbox = [];

          for (let [key, campo] of Object.entries(e.target)) {
            if (campo.name === "" || typeof campo.name === "undefined") {
            } else {
              if (campo.type === "checkbox") {
                objeto[campo.name] = checkbox;
                if (campo.checked === true) {
                  checkbox.push(campo.value);
                  objeto[campo.name] = checkbox;
                }
              } else {
                if (
                  campo.value !== "" &&
                  campo.value !== null &&
                  typeof campo.value !== "undefined"
                ) {
                  objeto[campo.name] = campo.value;
                }
              }
            }
          }
          this.store(objeto);
          break;
        case "destroy":
          this.destroy(id);
          break;
        case "update":
          var objeto = {};
          var checkbox = [];

          for (let [key, campo] of Object.entries(e.target)) {
            if (campo.name === "" || typeof campo.name === "undefined") {
            } else {
              if (campo.type === "checkbox") {
                objeto[campo.name] = checkbox;
                if (campo.checked === true) {
                  checkbox.push(campo.value);
                  objeto[campo.name] = checkbox;
                }
              } else {
                if (
                  campo.value !== "" &&
                  campo.value !== null &&
                  typeof campo.value !== "undefined"
                ) {
                  objeto[campo.name] = campo.value;
                }
              }
            }
          }
          this.update(id, objeto);

          break;

        default:
          break;
      }
    },
    dadosInputUpdateSelect(inputVar) {
      let splitInputvar = inputVar.split(".");
      let dados = this.dados;
      if (dados.success) {
        if (splitInputvar.length > 1) {
          for (let [key, value] of Object.entries(dados.success)) {
            var ret = value.data.filter((idDados) => {
              if (
                idDados[this.primaryKey] == this.modalIdUpdate[this.primaryKey]
              )
                return idDados;
            });
            try {
              if (splitInputvar.length == 2) {
                return ret[0][splitInputvar[0]][splitInputvar[1]];
              } else if (splitInputvar.length == 3) {
                return ret[0][splitInputvar[0]][splitInputvar[1]][
                  splitInputvar[2]
                ];
              } else if (splitInputvar.length == 4) {
                return ret[0][splitInputvar[0]][splitInputvar[1]][
                  splitInputvar[2]
                ][splitInputvar[3]];
              } else if (splitInputvar.length == 5) {
                return ret[0][splitInputvar[0]][splitInputvar[1]][
                  splitInputvar[2]
                ][splitInputvar[3]][splitInputvar[4]];
              }
            } catch (error) {
              return "";
            }
          }
        } else {
          for (let [key, value] of Object.entries(dados.success)) {
            var ret = value.data.filter((idDados) => {
              if (
                idDados[this.primaryKey] == this.modalIdUpdate[this.primaryKey]
              )
                return idDados;
            });
            return ret[0][inputVar];
          }
        }
      }
    },
    dadosInputUpdateCheckbox(value, tit, selectId, slc) {
      if (value) {
        if (value[0][tit]["valor"] && value[0][tit]["valor"].length !== 0) {
          if (value[0][tit]["valor"].indexOf(slc[selectId]) != -1) {
            return true;
          } else {
            return false;
          }
        } else {
          return false;
        }
      } else {
        return false;
      }
    },
    async genericApi(rota, method, data) {
      this.$store.state.routeGeneric = route(rota);

      let dados = await this.$genericRequestApi(method, data, rota);

      for (let [key, value] of Object.entries(dados.success)) {
        if (value.data) {
          this.selectDados = value.data;
        } else {
          this.selectDados = value;
        }
      }
    },
    setButton(obj, action, tituloModal) {
      this.typeButtonFooter = "submit";
      this.formUpdateId = "formSubmitTable";
      this.modelValue = "";
      this.selectModel = "";
      this.modalIdUpdate = "";
      this.modalBody = {};
      this.modalTitle = tituloModal;
      this.layoutModal = action;
      this.modalIdUpdate = obj;
      switch (action) {
        case "store":
          this.typeButtonFooter = "submit";
          this.modalBtnSubmit = "btn btn-primary text-white";
          this.modalBtnSalvar = true;
          this.colBtnModal = "col-6";
          this.loadModal = false;
          this.modalDadosInp = this.modalSlot.modalStore;
          this.modalBtnTitleSalvar = "Salvar";
          this.dataAction = action;

          break;
        case "show":
          this.modalBtnSubmit = "btn btn-info";
          this.modalBtnSalvar = false;
          this.colBtnModal = "col-12";
          this.modalDadosInp = this.modalSlot.modalShow;
          this.show(obj[this.primaryKey]);
          break;
        case "destroy":
          this.modalBtnSubmit = "btn btn-danger text-white";
          this.modalBtnSalvar = true;
          this.colBtnModal = "col-6";
          this.modalBtnTitleSalvar = "Sim";
          this.modalButtonId = "btn-excluir";
          this.loadModal = false;
          this.modalNameItem = obj[this.secondKey];
          this.dataId = obj[this.primaryKey];
          this.dataAction = action;
          break;
        case "update":
          this.typeButtonFooter = "submit";
          this.loadModal = false;
          this.modalBtnSubmit = "btn btn-primary text-white";
          this.modalBtnSalvar = true;
          this.colBtnModal = "col-6";
          this.modalDadosInp = this.modalSlot.modalUpdate;
          this.show(obj[this.primaryKey]);
          this.modalBtnTitleSalvar = "Atualizar";
          this.dataId = obj[this.primaryKey];
          this.dataAction = action;
          this.clearinputCheckbox();

          break;
        default:
          break;
      }
    },
    tratarText(valor) {
      return valor;
    },
    tratarDate(valor) {
      if (valor) {
        return moment(valor).format("DD/MM/yyyy H:mm");
      }
    },
    tratarImage(valor) {
      return valor;
    },
    tratarFloat(valor) {
      return valor;
    },
    tratarMonetario(valor) {
      return valor;
    },
    tratarBagde(t, valor) {
      if (t["data"]) {
        for (let [key, value] of Object.entries(t["data"])) {
          if (value["check"] == valor) {
            return (
              '<span class="' + value["class"] + '">' + value["msg"] + "</span>"
            );
          }
        }
      }
    },

    async show(id) {
      this.paginate = false;
      this.loadModal = true;
      this.dadosModal = await this.$showApi(id);

      if (this.dadosModal.success) {
        for (let [inptuKey, inputValue] of Object.entries(
          this.dadosModal.success
        )) {
          this.modalBody = inputValue;
          // aqui tenho que chamar o componente 'modal-component'
        }
      } else {
        this.$toast.open({
          message: this.dadosModal.errors,
          type: "danger",
          dismissible: true,
          position: "bottom-right",
        });
      }
      this.loadModal = false;
    },

    async store(data) {
      this.paginate = false;
      this.$store.state.loadingButton = true;
      this.dadosModal = await this.$storeApi(data);
      if (this.dadosModal.success) {
        this.indexApi();
        this.$toast.open({
          message: "Dados inseridos",
          type: "success",
          dismissible: true,
          position: "bottom-right",
        });
      }
      this.$store.state.loadingButton = false;
    },

    async update(id, data) {
      this.paginate = false;
      this.$store.state.loadingButton = true;
      this.dadosModal = await this.$updateApi(id, data);
      if (this.dadosModal.success) {
        this.indexApi();
        this.$toast.open({
          message: "Dados atualizados",
          type: "success",
          dismissible: true,
          position: "bottom-right",
        });
      }
      this.$store.state.loadingButton = false;
    },

    async destroy(id) {
      this.paginate = false;
      this.$store.state.loadingButton = true;
      this.dadosModal = await this.$destroyApi(id);
      if (this.dadosModal.success) {
        this.closeModal();
        this.perPage = 50;
        this.indexApi();
        //registro deletado com sucesso
        this.$toast.open({
          message: "Registro deletado com sucesso!",
          type: "success",
          dismissible: true,
          position: "bottom-right",
        });
      } else {
        this.$toast.open({
          message: this.dadosModal.errors,
          type: "danger",
          dismissible: true,
          position: "bottom-right",
        });
      }
      this.$store.state.loadingButton = false;
    },

    async btnSearch() {
      this.paginate = false;
      this.dados = await this.$indexApi(
        this.$store.state.routeIndex,
        this.perPage,
        this.data,
        this.search,
        this.paginate
      );
    },
    async paginacao(p) {
      if (p.url) {
        this.paginate = true;
        this.dados = await this.$indexApi(
          p.url,
          this.perPage,
          this.data,
          this.search,
          this.paginate
        );
      }
    },
    async ordenaCollum(e) {
      this.paginate = false;
      let ordenaVal = this.ordenaVal === "asc" ? "desc" : "asc";
      this.dados = await this.$indexApi(
        this.$store.state.routeIndex,
        this.perPage,
        this.data,
        this.search,
        this.paginate,
        e,
        ordenaVal
      );
      this.ordenaVal = ordenaVal;
    },
    closeModal() {
      const click = new Event("click");
      const btnCloseModal = document.getElementById("btnCloseModal");
      btnCloseModal.dispatchEvent(click);
    },

    clearinputCheckbox() {
      var uncheck = document.getElementsByTagName("input");
      for (var i = 0; i < uncheck.length; i++) {
        if (uncheck[i].type == "checkbox") {
          uncheck[i].checked = false;
        }
      }
    },
    async indexApi() {
      //this.indexApi();
      this.paginate = false;
      this.dados = await this.$indexApi(
        this.$store.state.routeIndex,
        this.perPage,
        this.data,
        this.search,
        this.paginate
      );
    },
  },
  mounted() {
    this.indexApi();
  },
  watch: {
    perPage() {
      this.indexApi();
    },
  },
};
</script>
<style scoped>
table {
  font-size: 11px;
}
table thead th {
  cursor: pointer;
}
.icons-thead {
  font-size: 13px;
  color: #777;
}
</style>

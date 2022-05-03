<template>
  <app-layout title="Usuários">
    <template #header>
      <h2 class="h4 font-weight-bold">Usuários</h2>
    </template>
    <card-page
      :headerTitulo="headerTitulo"
      :headerTituloValor="headerTituloValor"
      :headerbotao="headerbotao"
    >
      <Table
        :primaryKey="this.primaryKey"
        :secondKey="this.secondKey"
        :titulos="this.titulos"
        :buttons="this.buttons"
        :modalSlot="this.modalSlot"
        :headerTituloValor ="this.headerTituloValor"
      />
    </card-page>
  </app-layout>
</template>

<script>
import { defineComponent } from "vue";
import AppLayout from "@/Layouts/AppLayout.vue";
//import Welcome from "@/Jetstream/Welcome.vue";
import CardPage from "@/components/CardPage.vue";
import Table from "@/components/Table.vue";

export default defineComponent({
  components: {
    AppLayout,
    CardPage,
    Table,
  },
  data() {
    return {
      headerTitulo: true,
      headerTituloValor: "Usuários",
      headerbotao: true,
      primaryKey: "id",
      secondKey: "name",
      titulos: {
        0: { titulo: "ID", tipo: "text", var: "id" },
        1: { titulo: "Nome", tipo: "text", var: "name" },
        2: { titulo: "Email", tipo: "text", var: "email" },
        3: {
          titulo: "Data Criação Usuario",
          tipo: "date",
          var: "created_at",
        },
      },
      buttons: {
        0: {
          name: "Visualizar",
          iconeTipo: "visibility",
          disabled: false,
          action: "show",
          class: "btn btn-outline-info btn-sm",
          tituloModal: "Visualizar Usuário",
        },
        1: {
          name: "Editar",
          iconeTipo: "edit",
          disabled: false,
          action: "update",
          class: "btn btn-outline-secondary btn-sm",
          tituloModal: "Editar Usuário",
        },
        2: {
          name: "Excluir",
          iconeTipo: "delete",
          disabled: false,
          action: "destroy",
          class: "btn btn-outline-danger btn-sm",
          tituloModal: "Excluir Usuário",
        },
      },
      modalSlot: {
        modalShow: {
          0: { inputType: "text", inputVar: "name", inputTitulo: "Nome" },
          1: { inputType: "text", inputVar: "email", inputTitulo: "Email" },
          2: {
            inputType: "date",
            inputVar: "created_at",
            inputTitulo: "Data de Criação",
          },
          3: {
            inputType: "date",
            inputVar: "updated_at",
            inputTitulo: "Última Atualização",
          },
          4: {
            inputType: "array",
            inputVar: "p_users.p_perfils.nome",
            inputTitulo: "Perfil",
          },
        },
        modalStore: {
          0: {
            inputType: "text",
            inputVar: "name",
            inputTitulo: "Nome",
            inputRequired: true,
          },
          1: {
            inputType: "text",
            inputVar: "email",
            inputTitulo: "Email",
            inputRequired: true,
          },
          2: {
            inputType: "password",
            inputVar: "password",
            inputTitulo: "Senha",
            inputRequired: true,
          },
          3: {
            inputType: "password",
            inputVar: "password_confirmation",
            inputTitulo: "Confirme sua Senha",
            inputRequired: true,
          },
          4: {
            inputType: "checkbox",
            inputVar: "p_users.p_perfils.id",
            inputData: "perfil",
            inputTitulo: "Perfil",
            optionsSelectRoute: "indexPerfilsApi",
            optionsSelectMethod: "get",
            optionsSelectData: "",
            optionsSelectId: "id",
            optionsSelectVar1: "nome",
            optionsSelectVar2: "descricao",
            inputRequired: true,
          },
        },
        modalUpdate: {
          0: {
            inputType: "text",
            inputVar: "name",
            inputTitulo: "Nome",
            inputRequired: false,
          },
          1: {
            inputType: "text",
            inputVar: "email",
            inputTitulo: "Email",
            inputRequired: false,
          },

          2: {
            inputType: "password",
            inputVar: "password",
            inputTitulo: "Senha",
            inputRequired: false,
          },
          3: {
            inputType: "password",
            inputVar: "password_confirmation",
            inputTitulo: "Confirme sua Senha",
            inputRequired: false,
          },
          4: {
            inputType: "checkbox",
            inputVar: "p_users.p_perfils.id",
            inputData: "perfil",
            inputTitulo: "Perfil",
            optionsSelectRoute: "indexPerfilsApi",
            optionsSelectMethod: "get",
            optionsSelectData: "",
            optionsSelectId: "id",
            optionsSelectVar1: "nome",
            optionsSelectVar2: "descricao",
            inputRequired: false,
          },
        },
      },
    };
  },
  methods: {},
  async created() {
    this.$store.state.routeIndex = route("indexUsersApi");
    this.$store.state.routeShow = route("showUsersApi", "");
    this.$store.state.routeDestroy = route("destroyUsersApi", "");
    this.$store.state.routeUpdate = route("updateUsersApi", "");
    this.$store.state.routeStore = route("storeUsersApi");
  },
  mounted() {},
});
</script>

<template>
  <div>
    <div class="card card-default">
      <div class="card-header">
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <span>OAuth Clients</span>

          <a
            class="action-link"
            tabindex="-1"
            @click="showCreateClientForm"
          >Créer un nouveau client</a>
        </div>
      </div>

      <div class="card-body">
        <!-- Current Clients -->
        <p class="mb-0" v-if="clients.length === 0">Vous n'avez créez aucun client pour le moment.</p>

        <table class="table table-borderless mb-0" v-if="clients.length > 0">
          <thead>
            <tr>
              <th>Client ID</th>
              <th>Nom</th>
              <th>Secret</th>
              <th></th>
              <th></th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="client in clients" :key="client">
              <!-- ID -->
              <td style="vertical-align: middle;">{{ client.id }}</td>

              <!-- Name -->
              <td style="vertical-align: middle;">{{ client.name }}</td>

              <!-- Secret -->
              <td style="vertical-align: middle;">
                <code>{{ client.secret }}</code>
              </td>

              <!-- Edit Button -->
              <td style="vertical-align: middle;">
                <a class="action-link" tabindex="-1" @click="edit(client)">Editer</a>
              </td>

              <!-- Delete Button -->
              <td style="vertical-align: middle;">
                <a class="action-link text-danger" @click="destroy(client)">Supprimer</a>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Create Client Modal -->
    <div class="modal fade" id="modal-create-client" tabindex="-1" role="dialog">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h4 class="modal-title">Creer un client</h4>

            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
          </div>

          <div class="modal-body">
            <!-- Form Errors -->
            <div class="alert alert-danger" v-if="createForm.errors.length > 0">
              <p class="mb-0">
                <strong>Whoops!</strong> Something went wrong!
              </p>
              <br />
              <ul>
                <li v-for="error in createForm.errors" :key="error">{{ error }}</li>
              </ul>
            </div>

            <!-- Create Client Form -->
            <form role="form">
              <!-- Name -->
              <div class="form-group row">
                <label class="col-md-3 col-form-label">Nom</label>

                <div class="col-md-9">
                  <input
                    id="create-client-name"
                    type="text"
                    class="form-control"
                    @keyup.enter="store"
                    v-model="createForm.name"
                  />

                  <span class="form-text text-muted">Something your users will recognize and trust.</span>
                </div>
              </div>

              <!-- Redirect URL -->
              <div class="form-group row">
                <label class="col-md-3 col-form-label">Redirect URL</label>

                <div class="col-md-9">
                  <input
                    type="text"
                    class="form-control"
                    name="redirect"
                    @keyup.enter="store"
                    v-model="createForm.redirect"
                  />

                  <span class="form-text text-muted">Your application's authorization callback URL.</span>
                </div>
              </div>
            </form>
          </div>

          <!-- Modal Actions -->
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-dismiss="modal">Fermer</button>

            <button type="button" class="btn btn-primary" @click="store">Créer</button>
          </div>
        </div>
      </div>
    </div>

    <!-- Edit Client Modal -->
    <div class="modal fade" id="modal-edit-client" tabindex="-1" role="dialog">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h4 class="modal-title">Editer Client</h4>

            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
          </div>

          <div class="modal-body">
            <!-- Form Errors -->
            <div class="alert alert-danger" v-if="editForm.errors.length > 0">
              <p class="mb-0">
                <strong>Whoops!</strong> Something went wrong!
              </p>
              <br />
              <ul>
                <li v-for="error in editForm.errors" :key="error">{{ error }}</li>
              </ul>
            </div>

            <!-- Edit Client Form -->
            <form role="form">
              <!-- Name -->
              <div class="form-group row">
                <label class="col-md-3 col-form-label">Nom</label>

                <div class="col-md-9">
                  <input
                    id="edit-client-name"
                    type="text"
                    class="form-control"
                    @keyup.enter="update"
                    v-model="editForm.name"
                  />

                  <span class="form-text text-muted">Something your users will recognize and trust.</span>
                </div>
              </div>

              <!-- Redirect URL -->
              <div class="form-group row">
                <label class="col-md-3 col-form-label">Redirect URL</label>

                <div class="col-md-9">
                  <input
                    type="text"
                    class="form-control"
                    name="redirect"
                    @keyup.enter="update"
                    v-model="editForm.redirect"
                  />

                  <span class="form-text text-muted">Your application's authorization callback URL.</span>
                </div>
              </div>
            </form>
          </div>

          <!-- Modal Actions -->
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-dismiss="modal">Fermer</button>

            <button type="button" class="btn btn-primary" @click="update">Enregistrer</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  /*
   * The component's data.
   */
  data() {
    return {
      clients: [],

      createForm: {
        errors: [],
        name: "",
        redirect: "",
        confidential: true
      },

      editForm: {
        errors: [],
        name: "",
        redirect: ""
      }
    };
  },

  /**
   * Prepare the component (Vue 1.x).
   */
  ready() {
    this.prepareComponent();
  },

  /**
   * Prepare the component (Vue 2.x).
   */
  mounted() {
    this.prepareComponent();
  },

  methods: {
    /**
     * Prepare the component.
     */
    prepareComponent() {
      this.getClients();

      $("#modal-create-client").on("shown.bs.modal", () => {
        $("#create-client-name").focus();
      });

      $("#modal-edit-client").on("shown.bs.modal", () => {
        $("#edit-client-name").focus();
      });
    },

    /**
     * Get all of the OAuth clients for the user.
     */
    getClients() {
      axios.get("/oauth/clients").then(response => {
        this.clients = response.data;
      });
    },

    /**
     * Show the form for creating new clients.
     */
    showCreateClientForm() {
      $("#modal-create-client").modal("show");
    },

    /**
     * Create a new OAuth client for the user.
     */
    store() {
      this.persistClient(
        "post",
        "/oauth/clients",
        this.createForm,
        "#modal-create-client"
      );
    },

    /**
     * Edit the given client.
     */
    edit(client) {
      this.editForm.id = client.id;
      this.editForm.name = client.name;
      this.editForm.redirect = client.redirect;

      $("#modal-edit-client").modal("show");
    },

    /**
     * Update the client being edited.
     */
    update() {
      this.persistClient(
        "put",
        "/oauth/clients/" + this.editForm.id,
        this.editForm,
        "#modal-edit-client"
      );
    },

    /**
     * Persist the client to storage using the given form.
     */
    persistClient(method, uri, form, modal) {
      form.errors = [];

      axios[method](uri, form)
        .then(response => {
          this.getClients();

          form.name = "";
          form.redirect = "";
          form.errors = [];

          $(modal).modal("hide");
        })
        .catch(error => {
          if (typeof error.response.data === "object") {
            form.errors = _.flatten(_.toArray(error.response.data.errors));
          } else {
            form.errors = ["Something went wrong. Please try again."];
          }
        });
    },

    /**
     * Destroy the given client.
     */
    destroy(client) {
      axios.delete("/oauth/clients/" + client.id).then(response => {
        this.getClients();
      });
    }
  }
};
</script>

<style scoped>
.action-link {
  cursor: pointer;
}
</style>


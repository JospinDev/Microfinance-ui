<template>
  <Navbar />
  <Modal :isVisible="show_debloque_modal" @close="closeModal">
    <form class="form" @submit.prevent="">
      <span class="title">
        Débloquer le compte: <span>&nbsp; {{ compte }}</span>
      </span>
      <div class="content">
        <label for="pour">Deloquer pour:</label>
        <select id="pour" v-model="pour">
          <option value="" disabled>--------</option>
          <option value="retrait">Retrait</option>
          <option value="virement">Virement</option>
        </select>
        <label v-if="pour" for="peyant">Deloquer avec:</label>
        <select v-if="pour" name="peyant" id="peyant" v-model="peyant" required>
          <option value="" disabled>--------</option>
          <!-- <option v-if="pour == 'retrait'" value="cahier">Cahier</option> -->
          <option value="cheque">Remise chèque</option>
          <option v-if="pour === 'retrait'" value="quittance">Quittance</option>
          <option v-if="pour === 'virement'" value="ordre de virement interne">Virement interne</option>
          <option v-if="pour === 'virement'" value="ordre de virement externe">Virement externe</option>
        </select>
        <label v-if="peyant !== 'cahier'" for="numero">Numero:</label>
        <input v-if="peyant !== 'cahier'" type="text" v-model="code" id="numero" />
      </div>
      <button class="btn-modal" @click="handleDeblocage">Debloquer</button>
    </form>
  </Modal>
  <Modal :isVisible="show_select" @close="close">
    <form @submit.prevent="" class="form">
      <span class="title">Filtrage</span>
      <div class="content">
        <label>Est-il une personne moral: </label>
        <select v-model="personne_physique__isnull" class="trois">
          <option value="">------</option>
          <option value="true">Oui</option>
          <option value="false">Non</option>
        </select>
        <label>Solde superieur : </label>
        <input type="number" v-model="solde__gte" />
        <label>Solde inferieur : </label>
        <input type="number" v-model="solde__lte" />
        <label>Commune : </label>
        <input type="text" v-model="commune_filtre" />
        <label>Colline /Quartier :</label>
        <input type="text">
        <label>Date d'activitée: </label>
        <div class="inputRow">
          <input type="date" v-model="date_fin" />
          <input type="date" v-model="date_debut" />
        </div>

        <div v-show="personne_physique__isnull === 'false'">
          <label>Date de naissance: </label>
          <div class="inputRow">
            <div v-show="personne_physique__isnull === 'false'">
              <input type="date" v-model="personne_physique_gte" />
            </div>
            <div v-show="personne_physique__isnull === 'false'">
              <input type="date" v-model="personne_physique_lte" />
            </div>
          </div>
        </div>


        <div class="un">
          <div v-show="personne_physique__isnull === 'false'">
            <label>Residence : </label>
            <select v-model="personne_physique__residence" class="trois">
              <option value="">------</option>
              <option value="RESIDENT">Resident</option>
              <option value="NON+RESIDENT">Non resident</option>
            </select>
          </div>
          <div v-show="personne_physique__isnull === 'false'">
            <label> Sexe : </label>
            <select v-model="personne_physique__sexe" class="trois">
              <option value="">------</option>
              <option value="M">Masculin</option>
              <option value="F">Feminin</option>
              <option value="NA">Non appliquer</option>
              <option value="autre">Autre</option>
            </select>
          </div>
        </div>

        <div class="un" v-show="personne_physique__isnull === 'true'">
          <label>Date de création: </label>
          <div class="inputRow">
            <input type="date" v-model="creation_debut" />
            <input type="date" v-model="creation_fin" />
          </div>
          <label>Institution : </label>
          <select v-model="personne_morale__institution" class="trois">
            <option value="">------</option>
            <option value="INSTITUTION+PUBLIQUE">Institution publique</option>
            <option value="INSTITUTION+PRIVEE">Institution privée</option>
            <option value="INSTITUTION+FINANCIERE">Institution financiere</option>
            <option value="INSTITUTION+NON+FINANCIERE">
              Institution non financiere
            </option>
          </select>

          <!-- <label>Personne morale activite : </label>
          <select v-model="personne_morale__activite" class="trois">
            <option value="">------</option>
            <option value="Administration+publique">Administration publique</option>
            <option value="Secteur+priv%C3%A9">Secteur privée</option>
            <option value="Secteur+financier">Secteur finsnciere</option>
            <option value="Secteurs+non+financiers">Secteur non financiere</option>
            <option value="AUTRES">Autres</option>
          </select> -->
        </div>
        <div>
          <label>Activite personnel: </label>
          <select v-model="personne_physique__activite" class="trois">
            <option value="">------</option>
            <option value="AGRO-ELEVEUR">Agro Eleveur</option>
            <option value="COMMERCANT">Commercant</option>
            <option value="SALARIE">Salarie</option>
            <option value="INDUSTRIEL">Industriel</option>
            <option value="SANS">Sans</option>
          </select>
        </div>  
      </div>
      <button class="btn-modal" @click="searchLasta('')">Filtrage</button>
    </form>
  </Modal>
  <Modal :isVisible="show_duplicate" @close="closeModal">
    <div class="form">
      <span class="title">Creation du sous compte</span>
      <div class="content">
        <label>Telephone : </label>
        <input type="text" placeholder="votre numero" v-model="phone" />
        <label>Adresse : </label>
        <input type="text" placeholder="Mettre coline ou quartier" v-model="adresse" />
        <div class="inputColomn">
          <label>Province:</label>
          <!-- <select v-model="province">
            <option value="" disabled>-------</option>
            <option value="Bujumbura">Bujumbura</option>
            <option value="Buhumuza">Buhumuza</option>
            <option value="Burunga">Burunga</option>
            <option value="Butanyerera">Butanyerera</option>
            <option value="Gitega">Gitega</option>
          </select> -->
          <input type="text" v-model="province">
          <label for="commune">Commune:</label>
          <!-- <select v-model="commune">
            <option value="" disabled>-------</option>
            <option v-if="province === ''" value="" disabled>Province d'abord</option>
            <option v-if="province === 'Buhumuza'" value="Butaganzwa">Butaganzwa</option>
            <option v-if="province === 'Buhumuza'" value="Butihinda">Butihinda</option>
            <option v-if="province === 'Buhumuza'" value="Cankuzo">Cankuzo</option>
            <option v-if="province === 'Buhumuza'" value="Gisagara">Gisagara</option>
            <option v-if="province === 'Buhumuza'" value="Gisuru">Gisuru</option>
            <option v-if="province === 'Buhumuza'" value="Muyinga">Muyinga</option>
            <option v-if="province === 'Buhumuza'" value="Ruyigi">Ruyigi</option>

            <option v-if="province === 'Bujumbura'" value="Bubanza">Bubanza</option>
            <option v-if="province === 'Bujumbura'" value="Bukinanyana">Bukinanyana</option>
            <option v-if="province === 'Bujumbura'" value="Cibitoke">Cibitoke</option>
            <option v-if="province === 'Bujumbura'" value="Isare">Isare</option>
            <option v-if="province === 'Bujumbura'" value="Mpanda">Mpanda</option>
            <option v-if="province === 'Bujumbura'" value="Mugere">Mugere</option>
            <option v-if="province === 'Bujumbura'" value="Mugina">Mugina</option>
            <option v-if="province === 'Bujumbura'" value="Muhuta">Muhuta</option>
            <option v-if="province === 'Bujumbura'" value="Mukaza">Mukaza</option>
            <option v-if="province === 'Bujumbura'" value="Ntahangwa">Ntahangwa</option>
            <option v-if="province === 'Bujumbura'" value="Rwibaga">Rwibaga</option>

            <option v-if="province === 'Burunga'" value="Bururi">Bururi</option>
            <option v-if="province === 'Burunga'" value="Makamba">Makamba</option>
            <option v-if="province === 'Burunga'" value="Matana">Matana</option>
            <option v-if="province === 'Burunga'" value="Musongati">Musongati</option>
            <option v-if="province === 'Burunga'" value="Nyanza">Nyanza</option>
            <option v-if="province === 'Burunga'" value="Rumonge">Rumonge</option>
            <option v-if="province === 'Burunga'" value="Rutana">Rutana</option>

            <option v-if="province === 'Butanyerera'" value="Busoni">Busoni</option>
            <option v-if="province === 'Butanyerera'" value="Kayanza">Kayanza</option>
            <option v-if="province === 'Butanyerera'" value="Kiremba">Kiremba</option>
            <option v-if="province === 'Butanyerera'" value="Kirundo">Kirundo</option>
            <option v-if="province === 'Butanyerera'" value="Matongo">Matongo</option>
            <option v-if="province === 'Butanyerera'" value="Muhanga">Muhanga</option>
            <option v-if="province === 'Butanyerera'" value="Ngozi">Ngozi</option>
            <option v-if="province === 'Butanyerera'" value="Tangara">Tangara</option>

            <option v-if="province === 'Gitega'" value="Bugendana">Bugendana</option>
            <option v-if="province === 'Gitega'" value="Gishubi">Gishubi</option>
            <option v-if="province === 'Gitega'" value="Gitega">Gitega</option>
            <option v-if="province === 'Gitega'" value="Karusi">Karusi</option>
            <option v-if="province === 'Gitega'" value="Kiganda">Kiganda</option>
            <option v-if="province === 'Gitega'" value="Muramvya">Muramvya</option>
            <option v-if="province === 'Gitega'" value="Mwaro">Mwaro</option>
            <option v-if="province === 'Gitega'" value="Nyabihanga">Nyabihanga</option>
            <option v-if="province === 'Gitega'" value="Shombo">Shombo</option>
          </select> -->
          <input type="text" v-model="commune">
        </div>
      </div>
      <button class="btn-modal" @click="sousCompte(client.id)"> Creer</button>
    </div>
  </Modal>


  <!-- <Modal :isVisible="show_edit" @close="closeModal">
    <form @submit.prevent="updateAccount" class="form">
      <span class="title">Modifier le compte</span>
      <div class="content">
        <div v-if="editingAccount.type === 'physique'" class="inputColomn">
          <label>Nom:</label>
          <input type="text" v-model="editingAccount.first_name">
          <label for="commune">Prenom:</label>
          <input type="text" v-model="editingAccount.last_name">
        </div>
        <div v-else class="inputColomn">
          <label>Nom:</label>
          <input type="text" v-model="editingAccount.nom">
        </div>
        <div v-if="editingAccount.type === 'physique'" class="inputColomn">
          <div  class="inputColomn">
            <label for="proffesion">Proffesion:</label>
            <input type="text" v-model="editingAccount.profession" id="proffesion">
          </div>
          <div class="inputColomn">
            <label for="cni">CNI:</label>
            <input type="text" v-model="editingAccount.CNI" id="cni">
          </div>
        </div>
        <div v-if="editingAccount.type === 'morale'" class="inputColomn">
          <label for="genre">Institution:</label>
          <select name="genre" id="genre" v-model="editingAccount.institution">
              <option value="" disabled>------</option>
              <option value="1">Banque centrale</option>
              <option value="2">Banques commerciales</option>
              <option value="3">Institution de microfinance</option>
              <option value="4">Autres sociétés financières</option>
          </select>
          <div class="inputColomn">
            <label for="cni">NIF:</label>
            <input type="text" v-model="editingAccount.NIF" id="cni">
          </div>
        </div>
        <div class="">
          <label for="activite">Activite:</label>
          <select name="activite" v-model="editingAccount.activite">
            <option value="AGRO-ELEVEUR">Agro Eleveur</option>
            <option value="COMMERCANT">Commercant</option>
            <option value="SALARIE">Salarie</option>
            <option value="INDUSTRIEL">Industriel</option>
            <option value="SANS">Sans</option>
            <option value="AUTRES">Autres</option>
          </select>
          <div v-if="editingAccount.type === 'physique'" >
            <label  for="Residence">Residence:</label>
            <select name="Residence" v-model="editingAccount.residence">
              <option value="" disabled selected>-------</option>
              <option value="RESIDENT">Resident</option>
              <option value="NON RESIDENT">Non Resident</option>
            </select>
          </div>
        </div>
      </div>
      <button v-if="modifier" class="btn-modal" type="submit">
        Modifier le compte &#10003;
      </button>
    </form>
  </Modal> -->
  
  
  <Modal 
    :isVisible="show_create_account" 
    @close="closeModal"
  >
    <GuichetCompteIndividuel 
      v-if="show_personne_physique" 
      @done="appendNewAccount" 
      @cancel="closeModal" 
      :updatingAccount="editingAccount"
      />
      <GuichetCompteMorale 
      v-if="show_personne_morale" 
      @done="appendNewAccount" 
      @cancel="closeModal" 
      :updatingAccount="editingAccount"
    />
  </Modal>
  <Modal :isVisible="show_mandataire" @close="closeModal">
    <div class="form">
      <span class="title">Créer un mandataire du compte {{ this.compte.numero }}</span>
      <div class="content">
        <label for="numero">Nom:</label>
        <input type="text" v-model="nom" placeholder="Nom" />
        <label for="numero">Prenom:</label>
        <input type="text" v-model="prenom" placeholder="Prenom" />
        <label for="numero">Adresse:</label>
        <input type="text" v-model="adresse" placeholder="Adresse" />
        <label for="numero">CNI:</label>
        <input type="number" v-model="CNI" placeholder="CNI" />
        <label for="numero">Telephone:</label>
        <input type="number" v-model="telephone" placeholder="Telephone" />
        <label for="numero">Details:</label>
        <textarea type="text" v-model="details" placeholder="Details" ></textarea>
        <label for="numero">Photos:</label>
        <input type="file" @change="handleImageUpload" accept="image/*" required />
      </div>
      <button @click="postMandataire" class="btn-modal">Créer</button>
    </div>
  </Modal>
  <div class="container">
    <div class="subMenu-headers">
      <div class="row">
        <button class="btn retour" @click="goBack">&#10094;</button>
        <GuichetSubmenu @createAccount="newAccount" />
      </div>
      <div class="row">
        <button @click="show_select = true" class="btn">
          Filtrer
        </button>
        <SearchComponent :searchFunction="searchLasta" />
      </div>
    </div>
    <div class="accounts">
      <div class="aCont">
        <span>COMPTE:</span>
        <span>{{ clients?.results?.length || 0 }}</span>
      </div>
      <Account account_name='SOLDE TOTAL:' :account_money="clients?.totals?.balance" />
    </div>
    <section class="table">
      <table>
        <tr>
          <th>Id</th>
          <th>Compte</th>
          <th>Nom et Prenom</th>
          <!-- <th>Genre</th> -->
          <!-- <th>CNI/NIF</th> -->
          <th>Solde</th>
          <th>Telephone</th>
          <!-- <th>Dernière activité</th> -->
          <th>Options</th>
        </tr>
        <tr v-for="client in clients.results" :key="client.id" @dblclick="goToProfile(client)" class="clickable-row">
          <td>{{ client.id }}</td>
          <td>{{ client.numero }}</td>
          <td>{{ getAccountOwnerName(client) }}</td>
          <td>{{ money(client.balance) }}</td>
          <td>{{ client.telephone }}</td>
          <!-- <td>{{ client.last_activity }}</td> -->
          <td>
            <div class="menu-container">
              <i class="btn fa fa-ellipsis-v" @click="toggleOptions(client.id)">
                <div v-if="selectedClient === client.id" class="option-links">
                  <div v-if="client.is_active">
                    <!-- <i class="fa-solid fa-money-bill-transfer"></i> -->
                    <i class="option-link fa-solid fa-money-bill-transfer"
                      @click="goToProfile(client)"><span>Transactions</span></i>
                    <i class="option-link fa-solid fa-clock-rotate-left"
                      @click="goToHistory(client)"><span>Historique</span></i>
                    <i class="option-link fa-solid fa-clone" @click="modalDupliquer(client)"><span>SousCompte</span></i>
                    <!-- <i class="option-link fa-solid fa-lock-open" @click="setStore(client)"><span>Débloqué</span></i> -->
                    <i class="option-link fa-solid fa-lock" @click="setStore(client)"><span>Deblocage</span></i>
                    <i class="option-link fa-solid fa-pencil" @click="modifier(client)"><span>Modifier</span></i>
                    <i class="option-link fa-solid fa-users" @click="goToMandataire(client)"><span>Mandataire</span></i>
                    <i class="option-link fa-brands fa-buromobelexperte">
                      <span>Hist. transactions</span>
                      <div class="actions">
                        <i 
                          @click="navigateToRoute(
                            `/depots/${client.id}?name=${getAccountOwnerName(client)}&number=${client.numero}`
                          )" class="option-link fa-solid fa-arrow-up"
                        >
                          <span>Depots</span>
                        </i>
                        <i @click="navigateToRoute(`/retraits/${client.id}?number=${client.numero}`)" class="option-link fa-solid fa-arrow-down">
                          <span>Retraits</span>
                        </i>
                        <i 
                          @click="navigateToRoute(`/virements/${client.id}?number=${client.numero}`)"
                          class="option-link fa-solid fa-arrows-turn-right">
                          <span>Virements</span>
                        </i>
                      </div>
                    </i>
                    
                    <i v-if="client?.allowed_by" class="option-link fa-solid fa-sack-dollar">
                      <span>
                        2M+ 
                        <span class="valid">{{ client?.allowed_by }}</span>
                      </span>
                    </i>
                    <i v-else class="option-link fa-solid fa-sack-dollar" @click="allow2Mmore(client.id)">
                      <span>
                        Autoriser 2M+ 
                      </span>
                    </i>
                  </div>
                  <!-- <i v-else class="option-link fa-solid fa-power-off" @click="getActivation('activer_compte', client.id)">
                    <span>Activer</span>
                  </i>
                  <i v-if="client.is_active" class="desactiver fa-solid fa-power-off" @click="getActivation('desactiver_compte', client.id)">
                    <span>Désactiver</span>
                  </i> -->
                </div>
              </i>
            </div>
          </td>
        </tr>
      </table>
    </section>
  </div>
</template>

<script>
import Navbar from "@/components/Navbar.vue";
import Modal from "@/Overlays/Modal.vue";
import GuichetSubmenu from "@/components/GuichetSubmenu.vue";
import SearchComponent from "@/components/SearchComponent.vue";
import Account from '@/components/account.vue';
import GuichetCompteIndividuel from "../../components/Compte_individuel.vue";
import GuichetCompteMorale from "../../components/Compte_morale.vue";

export default {
  data() {
    return {
      clients: [],
      allClients: [],
      show_debloque_modal: false,
      show_mandataire: false,
      show_duplicate: false,
      pour: '',
      peyant: "",
      code: "",
      search: "",
      show_select: false,
      show_edit: false,
      numero: "",
      personne_physique_gte: "",
      personne_physique_lte: "",
      personne_physique__isnull: "",
      personne_physique__date_naissance__gte: "",
      personne_physique__date_naissance__lte: "",
      personne_physique__activite: "",
      personne_physique__residence: "",
      personne_physique__sexe: "",
      personne_morale__institution: "",
      personne_morale__activite: "",
      clean: "",
      personne_physique__isnull: '',

      //Editing account
      first_name: '',
      last_name:'',
      activite:'',
      residence:'',

      editingAccount:null,

      phone: '',
      adresse: '',
      telephone: '',
      colline: '',
      commune: '',
      commune_filtre: '',
      province: '',
      organisation: '',
      payante: '',
      document: null,
      photo: null,
      message: '',
      compte: null,
      name: '',
      date_debut: '',
      date_fin: '',
      creation_debut: '',
      creation_fin: '',
      client: '',
      id: '',
      show_create_account: false,
      show_personne_physique: false,
      show_personne_morale: false,
      microfinances: [],
      microfinance: '',
      nom: '',
      prenom: '',
      CNI: '',
      compte: '',
      solde__gte: '',
      solde__lte: '',
      details: '',
      selectedClient: null
    };
  },
  components: {
    Navbar,
    GuichetSubmenu,
    SearchComponent,
    Modal,
    Account,
    GuichetCompteIndividuel,
    GuichetCompteMorale
  },
  watch: {
    "$store.state.compte_active": {
      deep: true,
      handler(new_val) {
        this.setStore(new_val);
      },
    },
    pour(new_val){
      if(new_val === 'virement'){
        this.getAccountCreationPrice(
          ['Ordre Virement Interne', 'Ordre Virement Externe', 'Virement Permanent'],
          'virement'
        )
      }
    }
  },
  methods: {
    appendNewAccount(updating = null, account) {
      this.closeModal();
      if(updating) {
        this.getClients()
        return
      }
      this.clients.results.unshift(account);
      this.$store.state.clients = this.clients;
      // this.closeModal();
    },
    newAccount(account) {
      account === 'morale' ? this.show_personne_morale = true : this.show_personne_physique = true
      this.show_create_account = true
    },
    mandataire(client) {
      this.show_mandataire = true;
      this.compte = client
    },
    toggleOptions(clientId) {
      this.selectedClient === clientId ? this.selectedClient = null : this.selectedClient = clientId;
    },
    modalDupliquer(client) {
      this.show_duplicate = true;
      this.client = client
      this.phone = client.telephone;
      this.adresse = client.adresse;
      this.province = client.province;
      this.commune = client.commune
    },
    effacer() {
      this.personne_morale__institution = '';
    },
    close() {
      this.show_select = false;
    },
    searchLasta(keyword) {
      axios
        .get(
          keyword ? `/comptes/?search=${keyword}` :
          `/comptes/?commune__icontains=${this.commune_filtre}&balance__gte=${this.solde__gte}&balance__lte=${this.solde__lte}&created_at__gte=${this.creation_debut}&created_at__lte=${this.creation_fin}&personne_physique__date_naissance__gte=${this.personne_physique__date_naissance__gte}&personne_physique__date_naissance__lte=${this.personne_physique__date_naissance__lte}&last_activity__gte=${this.date_debut}&last_activity__lte=${this.date_fin}&adresse=${this.adresse}&personne_physique__activite=${this.personne_physique__activite}&personne_physique__residence=${this.personne_physique__residence}&personne_physique__sexe=${this.personne_physique__sexe}&personne_physique__isnull=${this.personne_physique__isnull}&personne_morale__institution=${this.personne_morale__institution}&personne_morale__activite=${this.personne_morale__activite}`
          
        ).then((response) => {
          this.clients = response.data;
          this.closeModal()
          this.$store.state.clients = this.clients;
        }).catch((error) => {
          this.displayErrorOrRefreshToken(error,()=> this.searchLasta(keyword));
        });
    },
    setStore(compte) {
      this.$store.state.compte_active = compte;
      (this.show_debloque_modal = true), (this.client = compte.id), (this.compte = compte.numero);
    },
    async getClients() {
      axios
        .get("comptes/")
        .then((response) => {
          this.clients = response.data;
          this.$store.state.clients = this.clients;
        }).catch((error) => {
          this.displayErrorOrRefreshToken(error, this.getClients);
        });
    },
    goToProfile(compte) {
      this.$store.state.compte_active = compte;
      this.$router.push({ name: "GProfile" });
    },
    goToHistory(compte) {
      this.$store.state.triger = 'client'
      this.$store.state.compte_active = compte;
      this.$router.push({ name: "HistoriqueClient", query: { 'numero': compte.numero } });
    },
    goToMandataire(compte) {
      this.$router.push({ name: "mandataire", query: { id: compte.id, numero: compte.numero } });
    },
    async sousCompte(id) {
      (this.$store.state.message.error = ""), (this.$store.state.message.success = "");
      const form = {
        telephone: this.phone,
        adresse: this.adresse,
        commune: this.commune,
        province: this.province,
      };
      axios
        .post(`comptes/${id}/creer_sous_compte/`, form)
        .then((response) => {
          this.$store.state.message.success = "Sous compte creer avec success";
          this.$store.state.clients.results.unshift(response.data);
          this.closeModal();
        }).catch((error) => {
          this.displayErrorOrRefreshToken(error, () => this.sousCompte(id));
        })
    },
    async handleDeblocage() {
      if (!this.peyant) {
        this.$store.state.message.error = 'Une action est obligatoire.'
        return
      }
      const data = {
        numero: this.code,
        unblock_for: this.peyant,
        compte: this.client,
      };
      try {
        const response = await axios.post(`deblocages/${this.pour === 'virement' ? 'for_virement/' : ''}`, data);
        this.$store.state.message.success = "Débloqué avec succès.";

        let accountIndex = this.clients.results.findIndex(cli => cli.id === this.client);
        if (accountIndex !== -1) {
          this.clients.results[accountIndex].deblocages.unshift(response.data);
          this.$store.state.clients.results[accountIndex].deblocages.unshift(response.data);
        }
        // this.$store.state.compte_active.deblocages.unshift(response.data);
        // this.$store.state.compte_active.deblocages.unshift(response.data);


        this.$nextTick(() => this.closeModal())
      } catch (error) {
        this.displayErrorOrRefreshToken(error, this.handleDeblocage);
      }
    },
    closeModal() {
      this.show_edit = false;
      this.show_select = false;
      this.peyant = "";
      this.code = "";
      this.client = "";
      this.compte = "";
      this.pour = '';
      this.show_duplicate = false;
      this.show_debloque_modal = false;
      this.show_personne_physique = false
      this.show_personne_morale = false
      this.show_create_account = false
      this.show_mandataire = false;
      this.personne_physique__isnull = ''
      this.solde__gte = ''
      this.solde__lte = ''
      this.commune = ''
      this.date_fin = ''
      this.date_debut = ''
      this.personne_physique_gte = ''
      this.personne_physique_lte = ''
      this.personne_physique__residence = ''
      this.personne_physique__sexe = ''
      this.creation_fin = ''
      this.creation_debut = ''
      this.personne_morale__institution = ''
      this.personne_morale__activite = ''
      this.personne_physique__activite = ''
    },
    modifier(compte) {
      // const comfirmation = confirm(`Vous voulez vraiment modifier le compte ${compte.numero}?`);
      // if (comfirmation) {
        const type = compte.personne_physique ? 'physique' : 'morale'
        const accID = type === 'physique' ? compte.personne_physique.id : compte.personne_morale.id
        const url = compte.personne_physique ? 'personne_physiques' : 'personne_morales'

        axios.get(`${url}/${accID}/`)
        .then((response) => {
          this.newAccount(type)
          this.$nextTick(()=>{
            this.editingAccount = { 
              ...response.data, 
              type : type,
              numero : compte.numero,
              compte_id : compte.id,
              url : `${url}/${accID}/`
            }
          })
        }).catch((error) => {
          this.displayErrorOrRefreshToken(error, () => this.getActivation(action, id));
        })

        this.editingAccount = compte.personne_physique
      // }
    },
    handlePdfUpload(event) {
      this.document = event.target.files[0];
    },
    handleImageUpload(event) {
      this.photo = event.target.files[0];
    },
    // async updateAccount() {
    //   try {
    //     await axios.put(
    //       this.editingAccount.url, 
    //       this.editingAccount
    //     )
    //     this.$store.state.message.success = `Le compte ${this.editingAccount.numero} a été modifié avec succès.`
        
    //     this.editingAccount = null
    //     this.closeModal()
    //     this.getClients()
    //   } catch (error) {
    //     this.displayErrorOrRefreshToken(error, this.updateAccount)
    //   }
    // },
    update(data){
      this.clients.results = this.clients.results.map(cli => {
        if (cli.id === data.id) return { ...cli, ...data }
        return cli
      })
    },
    allow2Mmore(id) {
      axios.get(`comptes/${id}/autoriser_retrait/`)
        .then((reponse) => {
          this.$store.commit(
            'setSuccess', 
            reponse?.data?.status
          )
          const itemIndex = this.clients.results.findIndex((item)=> item.id === id)
          if(itemIndex !== -1) {
            this.clients.results[itemIndex].allowed_by = this.$store.state.user.user.username
          }
        }).catch((error) => {
          this.displayErrorOrRefreshToken(error, () => this.allow2Mmore(id));
        })
    },
    getActivation(action, id) {
      axios.get(`comptes/${id}/${action}/`)
        .then((reponse) => {
          this.$store.commit('setSuccess', `Compte ${action === 'activer_compte' ? 'activé' : 'desactivé' } avec succès.`)
          this.update(reponse.data)
        }).catch((error) => {
          this.displayErrorOrRefreshToken(error, () => this.getActivation(action, id));
        })
    },
    navigateToRoute(link){
      this.$router.push(link)
    },
  },
  mounted() {
    // this.getMicrofinance();
    // if (this.$store.state.clients.length) {
    //   this.clients = this.$store.state.clients
    // } else {
      this.getClients()
    // }
  },
};
</script>

<style scoped>
.option-link.fa-solid.fa-sack-dollar {
  min-width: 120px;
}

.fa-buromobelexperte {
  position: relative;
}

.fa-buromobelexperte div {
  display: none;
  position: absolute;
  background: var(--primary);
  border: 1px solid #eee;
  z-index: 1000;
  left: -60%;
  top: 0;
  padding: 5px;
  border-radius: 10px;

  i{
    padding: 10px;
  }
}

.fa-buromobelexperte:hover div {
  display: block;
}
</style>
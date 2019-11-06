<template>
  <div class="dashboard">
      <!-- Navbar -->
    <v-toolbar flat color="blue" height="50">
    <router-link to="/">
    <v-toolbar-title style="color: white; font-size: 24px;"><strong>Grimoire</strong></v-toolbar-title>
    </router-link>
    <v-spacer></v-spacer>
    <div style="color: white;">
        <strong>Balance: </strong> {{this.$store.state.balance}} AR
    </div>
  </v-toolbar>
  <!-- main -->
    <div style="padding: 20px;">
        <v-layout row wrap>
            <v-flex xs12 md3>
                <div style="background-color: whitesmoke; padding: 10px; text-align: center; border: 1px solid rgb(0, 140, 255);">
                <!-- publish dialog box -->
                <div class="text-center">
                <v-dialog
                persistent
                v-model="publishDialog"
                width="500"
                >
                <template v-slot:activator="{ on }">
                    <v-btn v-on="on" block depressed color="blue" class="white--text">Publish a paper</v-btn>
                </template>
                <div style="text-align: right; background-color: white;">
                    <v-spacer></v-spacer>
                    <v-btn @click="publishDialog = false" icon small depressed>
                        <v-icon>close</v-icon>
                    </v-btn>
                </div>
                <v-card>
                    <div style="padding: 10px;">
                        <v-text-field
                        v-model="paperTitle"
                        label="Paper title"
                        required
                        color="blue"
                        maxlength="60"
                    ></v-text-field>
                    <br>
                    <input style="display: none;" type="file" @change="onFileSelected" ref="pdfInput" accept="application/pdf">
                    <v-btn @click="$refs.pdfInput.click()" small block depressed outline color="blue" class="white--text">Upload the paper in PDF format</v-btn>
                    <br>
                    <div style="text-align: center;">
                        <v-checkbox @change="enableBtn()" v-model="licenseCheckbox" label="I confirm that this paper is published under a Creative Commons license or under Open Access." color="blue">

                    </v-checkbox>
                    </div>
                    <br>
                    <p :hidden="plswait" style="color: red;">Please wait ...</p>
                    <p :hidden="txdone" style="color: green;">Done! Please wait some time for it to reflect on the blockchain.</p>
                    </div>
                    <v-divider></v-divider>

                    <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                        color="blue"
                        text
                        small
                        depressed
                        class="white--text"
                        :disabled="publishBtn"
                        @click="publishPaper"
                    >
                        Publish
                    </v-btn>
                    </v-card-actions>
                </v-card>
                </v-dialog>
            </div>
                <!-- end box -->
                    
                    <v-btn @click="aboutSecView()" block outline depressed color="blue" class="white--text">About</v-btn>
                    <div style="text-align: left;" :hidden="aboutSec">
                        <br>
                        <p>Grimoire is a Decentralized Academic Publishing Portal where you can publish and have free, perpetual access to academic research papers in a manner that has never been possible before thanks to Arweave's blockchain technology.
                        <br>
                        <br>
                        <strong>There are only two requirements:</strong>
                        </p>
                        <ol>
                            <li>Publish your papers under a <a href="https://creativecommons.org/licenses/">Creative Commons license</a> or under <a href="https://en.wikipedia.org/wiki/Open_access">Open Access</a>.</li>
                            <br>
                            <li>Make sure everything is well-written since publishing something on the permaweb isn't reversible. You can't delete your publications once you published them.</li>
                        </ol>
                    </div>
                </div>
                <br>
                <!-- Donate -->
                <div style="background-color: whitesmoke; padding: 10px; text-align: center; border: 1px solid rgb(0, 140, 255);">
                    <h4 style="color: rgb(0, 140, 255);">Donate AR to an academic publisher:</h4>
                    <h6 style="color: grey;">(Click on <strong>$</strong> to copy address)</h6>
                    <br>
                    <v-text-field
                        v-model="arAddress"
                        label="AR Address"
                        flat
                        color="blue"
                    ></v-text-field>
                    <v-text-field
                        v-model="donationAmount"
                        label="Amount in AR"
                        flat
                        color="blue"
                        @change="enableDonationBtn()"
                    ></v-text-field>
                    <v-btn :disabled="donationBtn" @click="DonateAR" small depressed round color="blue" class="white--text">Donate</v-btn>
                    <br>
                    <div style="color: rgb(0, 140, 255);">{{txInfo}}</div>
                </div>
                <br>
                <br>
            </v-flex>
            <br>
            <v-flex md1></v-flex>
            <br>
            <br>
            <v-flex xs12 md8>
                <div style="background-color: whitesmoke; border: 1px solid rgb(0, 140, 255);">
                    <h2 style="padding: 4px; text-align: center; color: grey;">{{txOwner.length}} Available papers:</h2>
                    <div class="paperSection">
                        <br>
                        <div>
                            <!-- search a paper -->
                            <v-text-field
                            label="Search a paper"
                            v-model="search"
                            color="blue"
                        ></v-text-field>
                        </div>
                        <br>
                        <div class="paperCard"
                        v-for="tx in filteredList"
                        v-bind:key="tx"
                        >
                            <v-layout row wrap>
                                <v-flex md6>
                                    <h3>{{tx.txTagsDic.value}}</h3>
                                    <h5 style="color: grey; padding-top: 5px;"><span style="color: rgb(0, 140, 255);">Published by: </span>{{tx.txOwner}}</h5>
                                </v-flex>
                                <v-flex md6>
                                    <div style="text-align: right;">
                                        <v-btn @click="dlPdf(tx.txData)" title="Download" icon small color="transparent" class="blue--text"><v-icon>cloud_download</v-icon></v-btn>
                                        <v-btn @click="donateClipBoard(tx.txOwner)" title="Donate AR to the author" icon small color="transparent" class="blue--text"><v-icon>attach_money</v-icon></v-btn>
                                    </div>
                                </v-flex>
                            </v-layout>
                        </div>
                        <br>
                        <br>
                    </div>
                </div>
            </v-flex>
        </v-layout>
    </div>
    <v-footer fixed>
      <v-spacer></v-spacer>
      <div>
        <strong style="color: gray;">&copy;{{ new Date().getFullYear() }} <a href="https://github.com/merwane/grimoire">MIT</a></strong>
      </div>
      <v-spacer></v-spacer>
    </v-footer>
  </div>
</template>

<script>
import Arweave from 'arweave/web';
const arweave = Arweave.init();

// pdf to base64 converter
const toBase64 = file => new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () => resolve(reader.result);
    reader.onerror = error => reject(error);
});

export default {
  name: 'Dashboard',
  data() {
    return {
      aboutSec: true,
      publishDialog: false,
      publishBtn: true,
      licenseCheckbox: false,
      selectedFile: null,
      selectedFileData: '',
      paperTitle: '',
      search: '',
      plswait: true,
      txdone: true,
      txOwner: [],
      donationBtn: true,
      arAddress: '',
      donationAmount: '',
      txInfo: ''
    }
  },
  beforeCreate() {
    if(this.$store.state.wallet == ''){
        alert('Please login with your AR wallet before accessing the dashboard!')
      this.$router.push({
          name: 'Home'
        })
    }
  },
  created() {
      // get all transaction ids
      let $pageData = this;
      async function getTransactions() {
          // query
          const txids = await arweave.arql({
            op: 'equals',
            expr1: 'App-Name',
            expr2: 'grimoireprod'
            });
            // get data
            txids.forEach(tx => {
                arweave.transactions.get(tx).then(async transaction => {
                    var txOwner = await arweave.wallets.ownerToAddress(transaction.get('owner'));
                    var txData = transaction.get('data', {decode: true, string: true});
                    var txTagsDic = {}
                    // eslint-disable-next-line
                    var txTags = transaction.get('tags').forEach(tag => {
                        // eslint-disable-next-line
                        var tagName = tag.get('name', {decode: true, string: true});
                        var tagValue = tag.get('value', {decode: true, string: true});
                        txTagsDic['value'] = tagValue;
                    });
                    // store on data
                    $pageData.txOwner.push({txOwner, txTagsDic, txData});
                });
            });
            
      }

      getTransactions();
        
  },
  computed: {
      filteredList() {
          return this.txOwner.filter(tx => {
              return tx.txTagsDic.value.toLowerCase().includes(this.search.toLowerCase())
          })
      }
  },
  methods: {
      // toggle about section
      aboutSecView(){
          this.aboutSec = !this.aboutSec; 
      },
      // select a pdf to publish
    async onFileSelected(event){
        this.selectedFile = event.target.files[0];
        this.selectedFileData = await toBase64(event.target.files[0]);
    },
    // publish paper
    async publishPaper(){
        this.plswait = false;
        this.publishBtn = true;
        let transaction = await arweave.createTransaction({
        data: Buffer.from(this.selectedFileData, 'utf8')
    }, this.$store.state.wallet);

    // add tags

    transaction.addTag('Content-Type', 'application/pdf');
    transaction.addTag('App-Name', 'grimoireprod');
    transaction.addTag('grimoire', 'post');
    transaction.addTag('Paper-Title', this.paperTitle)

    // sign tx
    await arweave.transactions.sign(transaction, this.$store.state.wallet);

    // post tx
    // eslint-disable-next-line
    const response = await arweave.transactions.post(transaction);
    this.plswait = true;
    this.txdone = false;
    },
    enableBtn(){
        if (this.paperTitle !== '' && this.selectedFile !== null && this.licenseCheckbox !== false) {
            this.publishBtn = false;
        } else {
            this.publishBtn = true;
        }
    },
    enableDonationBtn(){
        if (this.arAddress !== '' && this.donationAmount !== '') {
            this.donationBtn = false;
        } else {
            this.donationBtn = true;
        }
    },
    dlPdf(data){
        var link = document.createElement('a');
        link.href = data;
        link.setAttribute('download', 'paper.pdf');
        document.body.appendChild(link);
        link.click();
    },
    donateClipBoard(address){
        this.arAddress = address;
    },
    async DonateAR(){
        // load tx
        let transaction = await arweave.createTransaction({
        target: this.arAddress,
        quantity: arweave.ar.arToWinston(parseFloat(this.donationAmount))
        }, this.$store.state.wallet);
        // sign
        await arweave.transactions.sign(transaction, this.$store.state.wallet);
        this.txInfo = "Please wait..."
        const response = await arweave.transactions.post(transaction);
        if (response.status == 200){
            this.txInfo = "Done!"
        } else if (response.status == 400) {
            this.txInfo = "Invalid transaction."
        } else if (response.status == 500) {
            this.txInfo = "Error."
        }
    }
  },
  
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
a, a:visited{
  text-decoration: none;
  color: rgb(14, 100, 212);
}
.paperSection{
    padding: 5px;
}
.paperCard{
    padding: 8px;
    border: 1px solid rgb(0, 140, 255);
    margin-bottom: 10px;
}
</style>

<template>
  <div class="home">
    <div>
      <div class="titleBar">GRIMOIRE</div>
      <div class="subTitleBar">A Decentralized Academic Publishing Portal for the 21st century.</div>
    </div>
    <!-- log -->
    <div class="loginBtn">
      <input style="display: none;" type="file" @change="onFileSelected" ref="walletInput" accept="application/json">
      <v-btn @click="$refs.walletInput.click()" large depressed round color="blue" class="white--text">Login with your AR Wallet</v-btn>
      <br>
    </div>
    <v-footer fixed color="transparent">
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

export default {
  name: 'Home',
  data() {
    return {
      selectedFile: null,
      output: ''
    }
  },
  methods: {
     async onFileSelected(event){
      this.selectedFile = event.target.files[0];
      // read json file
      const reader = new FileReader();
      await new Promise((res, rej) => {
        reader.onload = res
        reader.onerror = rej
        reader.readAsText(this.selectedFile)
      });


    const jwk = JSON.parse(reader.result)

    this.$store.state.wallet = jwk;
    
    // init arweave
    const arweave = Arweave.init();

    // get address
    arweave.wallets.jwkToAddress(jwk).then((address) => {
    this.output = address;
    this.$store.state.address = address;

    // get balance
    arweave.wallets.getBalance(address).then((balance) => {
    let ar = arweave.ar.winstonToAr(balance);

    this.$store.state.balance = ar;    
});
    });
  
    // redirect to dashboard
    if (jwk !== null) {
      this.$router.push({
          name: 'Dashboard'
    })
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
.home{
  background: rgb(226,234,237);
background: linear-gradient(180deg, rgba(226,234,237,1) 0%, rgba(162,217,237,1) 100%);
  height: 100vh;
}
.titleBar{
  text-align: center;
  font-size: 40px;
  color: rgb(97, 96, 96);
  letter-spacing: 5px;
  padding-top: 35px;
}
.subTitleBar{
  text-align: center;
  font-size: 20px;
  color: rgb(97, 96, 96);
  padding-top: 15px;
}
.loginBtn{
  position: fixed;
  text-align: center;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>

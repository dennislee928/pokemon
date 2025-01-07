<template>
  <div id="app">
    <form @submit.prevent="sendVerificationEmail">
      <input type="text" v-model="email" placeholder="email" required />
      <input type="password" placeholder="password" required />
      <div
        class="cf-turnstile"
        data-sitekey="0x4AAAAAAA4w98rWzg6uqdQP"
        data-callback="turnstileCallback"
      ></div>
      <button type="submit" value="Submit">Sign up</button>
    </form>
    <div v-for="pokemon in allPokemon" :key="`${pokemon.name}-${pokemon.url}`">
      <PokemonCard :pokemon="pokemon" @pokemon-details="updatePokemonDetails" />
    </div>
  </div>
</template>

<script>
import emailjs from "emailjs-com";
import PokemonCard from "./components/PokemonCard.vue";

export default {
  components: {
    PokemonCard,
  },
  data() {
    return {
      allPokemon: [], // This will be filled with Pokémon data
      email: "", // 新增 email 資料綁定
    };
  },
  mounted() {
    this.fetchAllPokemon();
    this.loadTurnstile();
  },
  methods: {
    async fetchAllPokemon() {
      try {
        const response = await fetch(
          "https://pokeapi.co/api/v2/pokemon?limit=10"
        );
        const data = await response.json();
        this.allPokemon = data.results;
      } catch (error) {
        console.error("Failed to fetch Pokémon:", error);
      }
    },
    updatePokemonDetails(data) {
      // Update allPokemon with the details fetched
      this.allPokemon = data;
    },
    loadTurnstile() {
      const script = document.createElement("script");
      script.src = "https://challenges.cloudflare.com/turnstile/v0/api.js";
      script.defer = true;
      document.head.appendChild(script);
    },
    turnstileCallback(token) {
      console.log(`Challenge Success: ${token}`);
      // 這裡可以加入驗證 token 的邏輯
    },
    async sendVerificationEmail() {
      try {
        const verificationUrl = `https://pokemon-7u0.pages.dev/verify?email=${encodeURIComponent(
          this.email
        )}`;
        const emailContent = `
          <p>請點擊以下按鈕完成註冊：</p>
          <a href="${verificationUrl}" style="display: inline-block; padding: 10px 20px; background-color: #4CAF50; color: white; text-decoration: none;">驗證電子郵件</a>
        `;

        // 使用 email.js 發送郵件
        const response = await emailjs.send(
          "service_e1ia1w5",
          "template_okiwtxe",
          {
            to_email: this.email,
            message_html: emailContent,
          },
          "rtofkHKZrBzmgJ_2T"
        );

        if (response.status === 200) {
          console.log("Verification email sent successfully.");
        } else {
          console.error("Failed to send verification email.");
        }
      } catch (error) {
        console.error("Error sending verification email:", error);
      }
    },
  },
};
</script>
<style>
/* Ensure this is all valid CSS */
.pokemon-card {
  border: 1px solid #ccc;
  padding: 10px;
}
</style>

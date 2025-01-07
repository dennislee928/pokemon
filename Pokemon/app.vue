<template>
  <div id="app">
    <form @submit.prevent="sendVerificationEmail" class="form-container">
      <input
        type="text"
        v-model="email"
        placeholder="Email"
        required
        @input="validateEmail"
        class="input-field"
      />
      <span v-if="emailError" class="error">{{ emailError }}</span>

      <input
        type="password"
        v-model="password"
        placeholder="Password"
        required
        @input="validatePassword"
        class="input-field"
      />
      <span v-if="passwordError" class="error">{{ passwordError }}</span>

      <div
        v-if="!turnstileVerified"
        class="cf-turnstile"
        data-sitekey="0x4AAAAAAA4w98rWzg6uqdQP"
        data-callback="turnstileCallback"
      ></div>
      <button
        type="submit"
        value="Submit"
        :disabled="emailError || passwordError || !turnstileVerified"
        class="submit-button"
      >
        Sign up
      </button>
    </form>
    <div
      v-if="turnstileVerified"
      v-for="pokemon in allPokemon"
      :key="`${pokemon.name}-${pokemon.url}`"
    >
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
      allPokemon: [],
      email: "",
      password: "",
      emailError: "",
      passwordError: "",
      turnstileVerified: false,
    };
  },
  mounted() {
    this.fetchAllPokemon();
    this.loadTurnstile();
  },
  methods: {
    validateEmail() {
      const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      this.emailError = emailPattern.test(this.email)
        ? ""
        : "Invalid email format";
    },
    validatePassword() {
      const passwordPattern =
        /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{9,}$/;
      this.passwordError = passwordPattern.test(this.password)
        ? ""
        : "Password must be >8 characters, include uppercase, lowercase, number, and special character";
    },
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
      this.turnstileVerified = true;
    },
    async sendVerificationEmail() {
      if (this.emailError || this.passwordError || !this.turnstileVerified)
        return;

      try {
        const verificationUrl = `https://pokemon-7u0.pages.dev/verify?email=${encodeURIComponent(
          this.email
        )}`;
        const emailContent = `
          <p>請點擊以下按鈕完成註冊：</p>
          <a href="${verificationUrl}" style="display: inline-block; padding: 10px 20px; background-color: #4CAF50; color: white; text-decoration: none;">驗證電子郵件</a>
        `;

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
.form-container {
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  background-color: #f9f9f9;
}

.input-field {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.submit-button {
  width: 100%;
  padding: 10px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.submit-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.error {
  color: red;
  font-size: 0.9em;
  margin-bottom: 10px;
  display: block;
}

.pokemon-card {
  border: 1px solid #ccc;
  padding: 10px;
  margin-top: 10px;
}
</style>

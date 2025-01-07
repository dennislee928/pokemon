<template>
  <div id="app">
    <div v-if="!turnstileVerified" class="turnstile-fullscreen">
      <div
        class="cf-turnstile"
        data-sitekey="0x4AAAAAAA4w98rWzg6uqdQP"
        data-callback="turnstileCallback"
      ></div>
    </div>

    <div v-else>
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

        <button type="submit" class="submit-button" :disabled="!isFormValid">
          Sign up
        </button>
      </form>

      <div v-if="showPrompt" class="prompt-message">請前往您的email收信</div>

      <div
        v-for="pokemon in allPokemon"
        :key="`${pokemon.name}-${pokemon.url}`"
      >
        <PokemonCard
          :pokemon="pokemon"
          @pokemon-details="updatePokemonDetails"
        />
      </div>
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
      showPrompt: false,
    };
  },
  computed: {
    isFormValid() {
      return (
        this.email && this.password && !this.emailError && !this.passwordError
      );
    },
  },
  mounted() {
    this.fetchAllPokemon();
    this.loadTurnstile();

    // 檢查是否是從驗證郵件返回
    const urlParams = new URLSearchParams(window.location.search);
    if (urlParams.has("verified")) {
      alert("驗證成功！請登入");
      // 清除 URL 參數
      window.history.replaceState({}, document.title, window.location.pathname);
    }

    window.turnstileCallback = (token) => {
      console.log("Turnstile Callback Triggered:", token);
      this.turnstileVerified = true;
      console.log("Verification Status:", this.turnstileVerified);
    };
  },
  methods: {
    validateEmail() {
      const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      this.emailError = emailPattern.test(this.email)
        ? ""
        : "Invalid email format";
      console.log("Email valid:", !this.emailError);
    },
    validatePassword() {
      const passwordPattern =
        /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{9,}$/;
      this.passwordError = passwordPattern.test(this.password)
        ? ""
        : "Password must be >8 characters, include uppercase, lowercase, number, and special character";
      console.log("Password valid:", !this.passwordError);
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
      script.async = true;
      document.head.appendChild(script);
    },
    async sendVerificationEmail() {
      if (this.emailError || this.passwordError) return;

      try {
        const verificationUrl = `https://covid-19-data.p.rapidapi.com/country/code?format=json&code=it`;
        const emailContent = `
          <p>親愛的使用者您好,</p>
          <p>請點擊下方按鈕完成驗證：</p>
          <button onclick="fetch('${verificationUrl}', {
            method: 'GET',
            headers: {
              'X-RapidAPI-Key': 'YOUR_RAPID_API_KEY',
              'X-RapidAPI-Host': 'covid-19-data.p.rapidapi.com'
            }
          }).then(() => {
            window.location.href = 'https://pokemon-7u0.pages.dev/?verified=true';
          })" style="display: inline-block; padding: 10px 20px; background-color: #4CAF50; color: white; text-decoration: none; border: none; cursor: pointer;">
            驗證
          </button>
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
          this.showPrompt = true;
          setTimeout(() => {
            this.showPrompt = false;
          }, 5000);
        } else {
          console.error("Failed to send verification email.");
        }
      } catch (error) {
        console.error("Error sending verification email:", error);
      }
    },
  },
  beforeDestroy() {
    delete window.turnstileCallback;
  },
};
</script>

<style>
.turnstile-fullscreen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f5f5f5;
  z-index: 1000;
}

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

.prompt-message {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  background-color: #4caf50;
  color: white;
  padding: 15px 30px;
  border-radius: 4px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  z-index: 1000;
  animation: fadeInOut 5s ease-in-out;
}

@keyframes fadeInOut {
  0% {
    opacity: 0;
  }
  10% {
    opacity: 1;
  }
  90% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}
</style>

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
      <form @submit.prevent="handleSignUp" class="form-container">
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

    // 檢查是否是驗證請求
    const urlParams = new URLSearchParams(window.location.search);
    if (urlParams.has("email") && urlParams.has("token")) {
      alert("email驗證成功！這個時候才fetch註冊api");
      fetch(
        "https://covid-19-data.p.rapidapi.com/country/code?format=json&code=it",
        {
          method: "GET",
          headers: {
            "X-RapidAPI-Key":
              "6a3dbc5b0dmsh65f22b45a5928dbp1a69f1jsn62d8d50826d3",
            "X-RapidAPI-Host": "covid-19-data.p.rapidapi.com",
          },
        }
      ).then(() => {
        alert("註冊成功！請登入");
        // 清除 URL 參數
        window.history.replaceState(
          {},
          document.title,
          window.location.pathname
        );
      });
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
    handleSignUp() {
      emailjs
        .send(
          "service_e1ia1w5",
          "template_okiwtxe",
          {
            to_email: this.email,
            message_html: `
            <p>親愛的使用者您好,</p>
            <p>請點擊下方按鈕完成驗證：</p>
            <a href='https://pokemon-7u0.pages.dev/verify?email=${this.email}' 
               style='display: inline-block; padding: 10px 20px; background-color: #4CAF50; color: white; text-decoration: none; border-radius: 4px;'>
              驗證
            </a>
          `,
          },
          "rtofkHKZrBzmgJ_2T"
        )
        .then(() => {
          alert("請前往您的註冊信箱驗證");
        })
        .catch((error) => {
          console.error("Error sending email:", error);
        });
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

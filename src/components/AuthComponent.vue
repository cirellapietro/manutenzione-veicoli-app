<template>
  <div class="auth-container">
    <div v-if="!user" class="login-options">
      <h2>Accedi a Manutenzione Veicoli</h2>
      
      <div class="social-logins">
        <button @click="signInWithGoogle" class="btn-google">
          Accedi con Google
        </button>
        
        <button @click="showEmailLogin = true" class="btn-email">
          Accedi con Email
        </button>
      </div>
      
      <div v-if="showEmailLogin" class="email-login">
        <input v-model="email" type="email" placeholder="La tua email">
        <input v-model="password" type="password" placeholder="La tua password">
        <button @click="signInWithEmail">Accedi</button>
      </div>
    </div>
    
    <div v-else class="user-profile">
      <h2>Benvenuto, {{ user.email }}</h2>
      <button @click="signOut">Esci</button>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import { supabase } from '../lib/supabaseClient';

export default {
  name: 'AuthComponent',
  setup() {
    const user = ref(null);
    const showEmailLogin = ref(false);
    const email = ref('');
    const password = ref('');

    onMounted(() => {
      checkAuthState();
    });

    const checkAuthState = async () => {
      const { data: { session } } = await supabase.auth.getSession();
      user.value = session?.user || null;
    };

    const signInWithGoogle = async () => {
      const { error } = await supabase.auth.signInWithOAuth({
        provider: 'google',
      });
      
      if (error) {
        console.error('Errore accesso Google:', error.message);
      }
    };

    const signInWithEmail = async () => {
      const { data, error } = await supabase.auth.signInWithPassword({
        email: email.value,
        password: password.value,
      });
      
      if (error) {
        console.error('Errore accesso:', error.message);
      } else {
        user.value = data.user;
      }
    };

    const signOut = async () => {
      const { error } = await supabase.auth.signOut();
      if (error) {
        console.error('Errore logout:', error.message);
      } else {
        user.value = null;
      }
    };

    supabase.auth.onAuthStateChange((event, session) => {
      user.value = session?.user || null;
    });

    return {
      user,
      showEmailLogin,
      email,
      password,
      signInWithGoogle,
      signInWithEmail,
      signOut,
    };
  },
};
</script>

<style scoped>
.auth-container {
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
}

.login-options {
  text-align: center;
}

.social-logins {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin: 20px 0;
}

.btn-google, .btn-email {
  padding: 12px;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
}

.btn-google {
  background-color: #fff;
  color: #757575;
  border: 1px solid #ddd;
}

.btn-email {
  background-color: #f1f1f1;
  color: #333;
}

.email-login {
  margin-top: 20px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #f9f9f9;
}

.email-login input {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.email-login button {
  width: 100%;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.user-profile {
  text-align: center;
}
</style>

<template>
  <div>
    <div class="alert-container" v-if="alert.visible">
      <a-alert
        :message="alert.message"
        :type="alert.type"
        show-icon
        closable
        @close="alert.visible = false"
      />
    </div>

    <div class="login-container">
      <a-card class="login-card">
        <a-typography-title :level="2" style="text-align: center; margin-bottom: -10px; color: #002776;">
          Welcome to Pizza-Pampano 🍕
        </a-typography-title>
        <a-typography-title :level="4" style="text-align: center; margin-bottom: 30px; color: #e31837;">
          Login to your account
        </a-typography-title>


        <a-form
          :model="formState"
          name="basic"
          autocomplete="off"
          @finish="onFinish"
          @finishFailed="onFinishFailed"
        >
          <a-form-item
            label="USERNAME"
            name="username"
            :rules="[{ required: true, message: 'Please input your username!' }]"
            :labelCol="{ span: 24 }"
            :wrapperCol="{ span: 24 }"
          >
            <template #label>
              <span style="margin-bottom: 2px; display: inline-block;">USERNAME</span>
            </template>
            <a-input class="inputs" v-model:value="formState.username" />
          </a-form-item>


          <a-form-item
            label="PASSWORD"
            name="password"
            :rules="[{ required: true, message: 'Password must be 8-11 characters!' }]"
            :labelCol="{ span: 24 }"
            :wrapperCol="{ span: 24 }"
            style="margin-bottom: 5px;"
          >
            <template #label>
              <span style="margin-bottom: 2px; display: inline-block;">PASSWORD</span>
            </template>
            <a-input-password class="inputs" v-model:value="formState.password" />
          </a-form-item>

          <a-form-item name="remember">
            <a-checkbox v-model:checked="formState.remember">Remember me</a-checkbox>
          </a-form-item>

          <a-form-item>
            <a-button class="button" type="primary" html-type="submit" block @click="handleLogin">Login</a-button>
          </a-form-item>
        </a-form>
      </a-card>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { reactive } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();


interface FormState {
  username: string;
  password: string;
  remember: boolean;
}

const formState = reactive<FormState>({
  username: '',
  password: '',
  remember: true,
});

const alert = reactive({
  visible: false,
  message: '',
  type: 'success' as 'success' | 'error'
});

const handleLogin = () => { 
  alert.message = '';
  alert.type = 'error';

  if (!formState.username || !formState.password) {
    alert.message = 'Error: Please enter both username and password!';
  } else if (formState.password.length < 8 || formState.password.length > 15) {
    alert.message = 'Error: Password must be 8-11 characters long!';
  } else {
    alert.message = 'Login Successful!';
    alert.type = 'success';
    setTimeout(() => {
      router.push('/store');
    }, 1000);
  }

  alert.visible = true;

  setTimeout(() => {
    alert.visible = false;
  }, 3000);
};

const onFinish = () => {
  handleLogin();
};

const onFinishFailed = () => {
  alert.message = 'Error: Please check the form fields!';
  alert.type = 'error';
  alert.visible = true;

  setTimeout(() => {
    alert.visible = false;
  }, 3000);
};
</script>

<style scoped>
.alert-container {
  position: fixed;
  top: 40px;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  max-width: 400px;
  z-index: 999;
}

.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: radial-gradient(circle, #fff5f5, #ffe1e1, #ffd6d6);
  padding: 20px;
}

.login-card {
  width: 100%;
  max-width: 480px;
  padding: 40px;
  border-radius: 16px;
  background: #ffffff;
  box-shadow: 0 8px 30px rgba(227, 24, 55, 0.3);
  border: 2px solid #1899e3;
}

.login-card h2,
.login-card h4 {
  color: #002776;
  font-weight: 700;
}

.inputs {
  height: 40px;
  border-radius: 8px;
  border: 1px solid #ccc;
  padding-left: 10px;
}

.button {
  background-color: #e31837;
  color: white;
  font-weight: bold;
  font-size: 16px;
  border-radius: 8px;
  border: none;
  transition: background 0.3s;
}

.button:hover {
  background-color: #002776;
  color: white;
}

a-checkbox {
  color: #002776;
}
</style>

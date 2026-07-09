<template>
    <div class="setup-2fa-container">
        <div class="setup-form-container">
            <h3>Setup Google Authenticator</h3>
            
            <div v-if="successMsg" class="success-box">
                <p>{{ successMsg }}</p>
                <router-link to="/" class="btn">Go Home</router-link>
            </div>

            <div v-else-if="qrCodeUrl">
                <p>1. Scan this QR code with your Google Authenticator app</p>
                <img :src="qrCodeUrl" alt="QR Code" style="margin: 20px 0; max-width: 200px;" />
                
                <p>2. Enter the 6-digit code to verify and enable 2FA</p>
                <div class="form-group">
                    <input type="text" class="form-control" placeholder="6-digit code" v-model="token" />
                </div>
                
                <div v-if="errorMsg" class="error-box">
                    <p>{{ errorMsg }}</p>
                </div>

                <div class="form-group">
                    <button @click="enable2FA" class="btn">Enable 2FA</button>
                </div>
            </div>

            <div v-else>
                <button @click="generate2FA" class="btn">Start Setup</button>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import { mapState } from 'vuex';

export default {
    name: 'Setup2FA',
    data() {
        return {
            qrCodeUrl: '',
            secret: '',
            token: '',
            errorMsg: '',
            successMsg: ''
        };
    },
    computed: {
        ...mapState(['user'])
    },
    methods: {
        async generate2FA() {
            try {
                const res = await axios.post('/auth/2fa/generate', { email: this.user.user_email });
                this.qrCodeUrl = res.data.qrCodeUrl;
                this.secret = res.data.secret;
            } catch (err) {
                this.errorMsg = "Failed to generate 2FA setup.";
            }
        },
        async enable2FA() {
            try {
                const res = await axios.post('/auth/2fa/enable', {
                    email: this.user.user_email,
                    secret: this.secret,
                    token: this.token
                });
                if (res.data.success) {
                    this.successMsg = "2FA has been successfully enabled!";
                    this.errorMsg = '';
                }
            } catch (err) {
                this.errorMsg = "Invalid token. Please try again.";
            }
        }
    },
    created() {
        if (!this.user) {
            this.$router.push('/login');
        }
    }
}
</script>

<style scoped>
.setup-2fa-container {
    padding: 2rem 9%;
}

.setup-form-container {
    background-color: #fff;
    padding: 2rem;
    border-radius: .5rem;
    box-shadow: 0 1rem 1rem rgba(0, 0, 0, 0.05);
    border: 0.1rem solid rgba(0, 0, 0, 0.2);
    text-align: center;
    max-width: 50rem;
    margin: 0 auto;
}

.setup-form-container h3 {
    font-size: 2rem;
    color: #130f40;
    margin-bottom: 2rem;
}

.setup-form-container p {
    font-size: 1.5rem;
    color: #666;
}

.form-control {
    margin: 1rem auto;
    border-radius: .5rem;
    background: #f7f7f7;
    padding: 1rem 1.2rem;
    font-size: 1.6rem;
    color: #130f40;
    width: 80%;
    border: none;
    text-align: center;
}

.btn {
    display: inline-block;
    padding: .8rem 3rem;
    font-size: 1.7rem;
    border-radius: .5rem;
    border: .2rem solid #27ae60;
    color: #27ae60;
    cursor: pointer;
    background: none;
    margin-top: 1rem;
}

.btn:hover {
    background: #27ae60;
    color: #fff;
}

.error-box p {
    color: red;
    font-size: 1.4rem;
}

.success-box p {
    color: green;
    font-size: 1.6rem;
    margin-bottom: 2rem;
}
</style>

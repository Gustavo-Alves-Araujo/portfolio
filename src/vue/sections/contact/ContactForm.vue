<template>
    <form @submit.prevent="submitForm" id="contact-form">
        <div class="row contact-form-row align-items-stretch">
            <!-- Feedback Alert -->
            <div class="col-12 mb-1" v-if="alertStatus">
                <Alert :type="alertStatus.type" :message="alertStatus.message" />
            </div>

            <!-- Left Column -->
            <div class="col-xl-6">
                <div class="form-group input-group">
                    <span class="input-group-text input-group-attach"><i class="fa fa-signature" /></span>
                    <input v-model="form.name" class="form-control" type="text" placeholder="Nome *" required />
                </div>

                <div class="form-group input-group">
                    <span class="input-group-text input-group-attach"><i class="fa fa-envelope" /></span>
                    <input v-model="form.email" class="form-control" type="email" placeholder="E-mail *" required />
                </div>

                <div class="form-group input-group">
                    <span class="input-group-text input-group-attach"><i class="fa fa-pen-to-square" /></span>
                    <input v-model="form.subject" class="form-control" type="text" placeholder="Assunto *" required />
                </div>
            </div>

            <!-- Right Column -->
            <div class="col-xl-6">
                <div class="form-group form-group-textarea mb-md-0">
                    <textarea v-model="form.message" class="form-control" placeholder="Mensagem *" required></textarea>
                </div>
            </div>

            <!-- Bottom Column -->
            <div class="col-12 text-center mt-3 mt-lg-4">
                <button class="btn btn-primary btn-xl" type="submit"
                    :class="{ disabled: submitStatus === SubmitStatus.SENDING }">
                    <i class="fa fa-envelope me-1" /> Enviar Mensagem
                </button>
            </div>
        </div>
    </form>
</template>

<script setup>
import { ref, computed } from 'vue';
import axios from 'axios';
import Alert from "../../widgets/Alert.vue";

const url = import.meta.env.VITE_BACKEND_URL

const form = ref({
    name: '',
    email: '',
    subject: '',
    message: ''
});

const SubmitStatus = {
    ENABLED: "enabled",
    SENDING: "sending",
    SENT: "sent",
    ERROR: "error"
};

const submitStatus = ref(SubmitStatus.ENABLED);

const submitForm = async () => {
    if (submitStatus.value === SubmitStatus.SENDING) return;

    submitStatus.value = SubmitStatus.SENDING;

    try {
        await axios.post(url + '/send-email', {
            email: form.value.email,
            subject: form.value.subject,
            message: form.value.message
        });

        submitStatus.value = SubmitStatus.SENT;
        form.value = { name: '', email: '', subject: '', message: '' }; // Clear form
    } catch (error) {
        submitStatus.value = SubmitStatus.ERROR;
    }
};

const alertStatus = computed(() => {
    switch (submitStatus.value) {
        case SubmitStatus.SENT:
            return { type: 'success', message: 'Mensagem enviada com sucesso!' };
        case SubmitStatus.ERROR:
            return { type: 'danger', message: 'Erro ao enviar mensagem. Tente novamente.' };
        default:
            return null;
    }
});
</script>
<style lang="scss" scoped>
@import "/src/scss/_theming.scss";

$form-input-background-color: lighten($light-3, 20%);
$form-input-placeholder-color: $light-5;

$form-input-border-color: $light-2;
$form-input-border-color-focus: lighten($primary, 5%);

$form-input-group-background-color: $light-1;
$form-input-group-font-color: $headings-color;

input,
textarea {
    padding: 1rem;
    font-family: $headings-font-family;

    background-color: $form-input-background-color;
    color: $dark;
    border: 2px solid $form-input-background-color;
    border-radius: 0;

    &:focus {
        border: 2px solid $form-input-border-color-focus;
    }
}

.input-group {
    @include generate-dynamic-styles-with-hash((xxxl: (margin-bottom: 0.8rem),
            sm: (margin-bottom: 0.4rem)));

    border: 2px solid $form-input-border-color;
}

.input-group-text {
    border: none;
    border-right: 2px solid $form-input-border-color;
    min-width: 60px;
    background-color: $form-input-group-background-color;

    border-radius: 0;
    text-align: center;

    i {
        color: $form-input-group-font-color;
        margin: 0 auto;
    }
}

input {
    @include generate-dynamic-styles-with-hash((xxxl: (height: auto),
            xl: (height: max(50px, 6.5vh)),
            lg: (height: max(40px, 4.5vh)),
            sm: (height: 40px, font-size: 0.8rem),
        ))
}

textarea {
    @include generate-dynamic-styles-with-hash((xxxl: (height: 218px),
            lg: (height: 200px),
            sm: (height: 150px, font-size: 0.8rem)));

    border: 2px solid $form-input-border-color;
}

::-webkit-input-placeholder {
    @include generate-dynamic-styles-with-hash((xxxl: (font-size: 1.1rem),
            lg: (font-size: 0.9rem)));

    font-family: $headings-font-family;
    color: $light-5;
}
</style>

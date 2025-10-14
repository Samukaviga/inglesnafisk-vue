<template>
  <form class="flex flex-col gap-3 w-full" @submit.prevent="handleSubmit">
    <Input v-model="registerForm.name" placeholder="Nome completo" required aria-label="Nome completo"
      :error="errors.name" />

    <Input v-model="registerForm.mobile_phone" type="tel" placeholder="WhatsApp / Celular" required
      v-mask="'(##) #####-####'" aria-label="WhatsApp" :error="errors.mobile_phone" />

    <Input v-model="registerForm.email" type="email" placeholder="E-mail" required aria-label="E-mail"
      :error="errors.email" />

    <Input v-model="registerForm.date_of_birth" type="tel" placeholder="Data de nascimento" required
      v-mask="'##/##/####'" aria-label="Data de nascimento" :error="errors.date_of_birth" />

    <Select v-model="registerForm.course" :placeholder="'Selecione o curso'" required :options="courseOptions"
      :error="errors.course || ''" />

    <Select v-model="registerForm.city" placeholder="Selecione uma Cidade *" required aria-label="Cidade"
      :options="citiesOptions" :error="errors.city">
    </Select>

    <div class="flex items-start gap-2 text-sm">
      <input id="aceito-termos" type="checkbox" required class="mt-1 accent-blue-800" />
      <label for="aceito-termos" class="select-none">
        Li e aceito os Termos de Uso e os Termos do Desconto.
      </label>
    </div>

    <Button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? 'Salvando...' : 'Confirmar Inscrição' }}
    </Button>
  </form>
</template>

<script setup lang="ts">
import { onMounted } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

import Input from '@/components/ui/Input.vue';
import Button from '@/components/ui/Button.vue';
import Select from '@/components/ui/Select.vue';
import type { RegisterForm } from '@/types/RegisterForm';
import { useContactStore } from '@/stores/contactStore';

const params = new URLSearchParams(window.location.search);

const router = useRouter();
import { computed, reactive, ref } from 'vue'

const errors = ref({})
const courseOptions = import.meta.env.VITE_COURSES.split(',').map(c => c.trim());
const citiesOptions = import.meta.env.VITE_ESCOLAS.split(',').map(c => c.trim());


function availableGroupedOptions() {
  return [
    {
      label: "Cursos Disponíveis",
      options: courseOptions.map(course => ({
        label: course,
        value: course,
      }))
    }
  ];
}


/*
const availableGroupedOptions = (index: number) => {
  const selected = registerForm.courses.map(c => c.course);

  return Object.entries(courseOptions).map(([category, items]) => ({
    label: category,
    options: items
        .filter(course => !selected.includes(course) || registerForm.courses[index].course === course)
        .map(course => ({
          label: course.trim(),
          value: course.trim()
        }))
  }));
}; */

const registerForm = reactive<RegisterForm>({
  name: '',
  mobile_phone: '',
  email: '',
  date_of_birth: '',
  course: '',
  city: '',
  utm_source: '',
  utm_medium: '',
  utm_campaign: '',
  utm_term: '',
  utm_content: '',
  gclid: '',
  fbclid: '',
  msclkid: '',
  referrer: '',
  landing_page: '',
});

onMounted(() => {
  const params = new URLSearchParams(window.location.search);

  registerForm.utm_source   = params.get('utm_source')   || '';
  registerForm.utm_medium   = params.get('utm_medium')   || '';
  registerForm.utm_campaign = params.get('utm_campaign') || '';
  registerForm.utm_term     = params.get('utm_term')     || '';
  registerForm.utm_content  = params.get('utm_content')  || '';
  registerForm.gclid        = params.get('gclid')        || '';
  registerForm.fbclid       = params.get('fbclid')       || '';
  registerForm.msclkid      = params.get('msclkid')      || '';

  // opcional: registrar a origem e página de destino
  registerForm.referrer = document.referrer || '';
  registerForm.landing_page = window.location.href;
});


const isSubmitting = ref(false);

const API_TOKEN = import.meta.env.VITE_API_TOKEN;
const API_URL = import.meta.env.VITE_API_URL;

async function handleSubmit() {

  try {

    isSubmitting.value = true;

    const response = await axios.post(`${API_URL}/registrations`, registerForm);

    console.log('Response:', response);

    const contact = response.data.data;

    const contactStore = useContactStore(contact.id);

    contactStore.setContact(contact);

    router.push(`/sucesso/${contact.id}`);
  } catch (error: any) {
    isSubmitting.value = false;

    alert("Oops! Encontramos um erro inesperado. Por favor, tente novamente.");
    if (error.response && error.response.status === 422) {
      errors.value = Object.fromEntries(
        Object.entries(error.response.data.errors).map(([key, messages]) => [key, messages[0]])
      )
    }
  }
}
</script>

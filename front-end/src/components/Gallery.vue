<template>
  <div class="bg-slate-200">
    <!-- Champ d'e-mail en dehors de la boucle v-for -->
    <div class="text-center mt-5 md:mt-16">
      <label for="email" class="block text-gray-700 font-custom mb-2">
        Adresse e-mail :
      </label>
      <input
        v-model="commonEmail"
        type="email"
        id="email"
        name="email"
        class="p-2 border rounded mx-auto"
      />
    </div>
    <div v-for="(item, index) in formList" :key="index" class="md:pt-5">
      <div class="text-center md:mt-24">
        <h1 class="font-custom text-2xl md:text-5xl">
          {{ item.formData.title }}
        </h1>
        <h3 class="text-2xl font-custom md:mt-5">
          {{ item.formData.question }}
        </h3>
      </div>

      <div
        v-if="item.visible"
        class="flex flex-col md:flex-row"
        data-aos="fade-down"
        data-aos-delay="450"
      >
        <div class="md:w-full lg:w-3/5 mt-5 md:mt-5 md:ml-5">
          <div class="relative w-full h-0" style="padding-bottom: 50%">
            <!-- Utilisation du rendu conditionnel pour vidéo ou image -->
            <video
              v-if="isVideo(item.videoUrl)"
              controls
              class="absolute inset-0 w-full h-full"
              :src="item.videoUrl"
              type="video/mp4"
            >
              Votre navigateur ne prend pas en charge la balise vidéo.
            </video>
            <img
              v-else
              class="absolute inset-0 w-full h-full object-cover"
              :src="item.videoUrl"
              alt="Image"
            />
          </div>
        </div>

        <div class="md:w-full lg:w-2/5 mt-1 md:pt-48 md:pl-6">
          <form
            v-if="item.visible"
            id="form"
            @submit.prevent="submitAllForms(index)"
            class="shadow-xl rounded-lg bg-slate-100 md:mr-5 pb-5 pl-5 pr-5 pt-5"
            :class="{ 'opacity-50': !item.canSubmit }"
          >
            <label
              for="message"
              class="block text-gray-700 font-custom mb-2 text-lg md:text-xl lg:text-2xl"
            >
              Donnez-nous votre avis :
            </label>
            <textarea
              v-model="item.formData.reponse"
              id="comment"
              class="rounded-lg border p-20 w-full text-lg md:text-xl lg:text-2xl"
            ></textarea>
          </form>
        </div>
      </div>
    </div>
    <!-- Bouton pour envoyer tous les formulaires -->
    <div>
      <div class="flex justify-center mt-8">
        <button
          @click="submitAllForms"
          class="mt-4 bg-blue-500 text-white font-custom rounded-lg px-4 py-2 text-lg md:text-xl lg:text-2xl"
        >
          Envoyer tous les formulaires
        </button>
      </div>
    </div>
  </div>
</template>

<!-- Le reste du code reste inchangé -->

<script setup>
import AOS from "aos";
import "aos/dist/aos.css";
import Swal from "sweetalert2";
import { ref } from "vue";
import formList from "@/data/formList.json";

AOS.init();

const commonEmail = ref(""); // Champ d'e-mail unique

const isVideo = (url) => {
  // Fonction pour vérifier si l'URL est une vidéo basée sur l'extension
  const lowerCaseUrl = url.toLowerCase();
  return lowerCaseUrl.endsWith(".mp4") || lowerCaseUrl.endsWith(".mov");
};

const submitAllForms = async (index) => {
  try {
    // Vérifier si le formulaire a déjà été soumis en consultant le localStorage
    const isFormSubmitted = localStorage.getItem("formSubmitted");

    if (isFormSubmitted == commonEmail.value) {
      // Afficher une alerte indiquant que le formulaire a déjà été soumis
      Swal.fire({
        icon: "error",
        title: "Impossible de répondre",
        text: "Vous avez déjà répondu à ce sondage. Vous ne pouvez pas répondre une deuxième fois."
      });
      return;
    }

    const responses = formList.map((item) => ({
      title: item.formData.title,
      reponse: item.formData.reponse,
      question: item.formData.question,
      email: commonEmail.value // Utilisez la valeur commune de l'e-mail
    }));

    const response = await fetch("http://localhost:3001/question", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify(responses)
    });
    console.log("Données envoyées :", JSON.stringify(responses));

    if (!response.ok) {
      throw new Error(
        `Erreur HTTP : ${response.status} - ${response.statusText}`
      );
    }

    const responseData = await response.json();

    if (responseData.error) {
      throw new Error(responseData.error);
    }

    // Désactiver tous les formulaires après l'envoi
    formList.forEach((item, i) => {
      item.canSubmit = commonEmail;
      item.visible = i === index ? false : item.visible; // Masquer le formulaire actuel
    });

    // Stocker l'état de l'envoi du formulaire dans le localStorage
    localStorage.setItem("formSubmitted", commonEmail.value);

    console.log("Réponse du backend :", responseData.message);

    // Afficher l'alerte pour l'envoi réussi
    Swal.fire({
      icon: "success",
      title: "Envoyé avec succès",
      text: "Vos réponses ont été soumises avec succès. Vous ne pouvez pas répondre à ce sondage une deuxième fois."
    });
  } catch (error) {
    if (error.message.includes("déjà répondu")) {
      // Afficher une alerte pour informer que l'utilisateur ne peut pas répondre une deuxième fois
      Swal.fire({
        icon: "warning",
        title: "Impossible de répondre",
        text: "Vous avez déjà répondu à ce sondage. Vous ne pouvez pas répondre une deuxième fois."
      });
    } else {
      console.error("Erreur lors de l'envoi des formulaires :", error.message);
    }
  }
};
</script>

<template>
    <div class="container">
        <h1>Email Preview and Download</h1>

        <div class="file12">
            <input type="file" @change="handleFileChange" accept=".eml" />
        </div>
        <div class="red3">
            <button @click="previewEmail" :disabled="!formattedEmailContent">Preview</button>
            <button @click="downloadEmail" :disabled="!emailContent">Download</button>
        </div>
        <div v-if="previewVisible" class="preview">
            <h2>Email Preview</h2>
            <pre v-if="formattedEmailContent">{{ formattedEmailContent }}</pre>
            <p v-else>No content to preview</p>
        </div>
    </div>
</template>

<script setup>
import { ref } from "vue";
import { EmlParser } from "eml-parser";

const emailContent = ref("");
const previewVisible = ref(false);
const formattedEmailContent = ref("");

const handleFileChange = (event) => {
    const file = event.target.files[0];
    if (!file) return;

    if (file.type !== "message/rfc822") {
        alert("Please select a .eml file");
        return;
    }

    const reader = new FileReader();
    reader.onload = () => {
        emailContent.value = reader.result;
    };
    reader.readAsText(file);
};

const previewEmail = () => {
    previewVisible.value = true;
    const eml = emailContent.value;

    const parser = new EmlParser(eml);
    parser
        .getEmailBodyHtml()
        .then((htmlString) => {
            formattedEmailContent.value = htmlString;
            console.log("Email preview:", htmlString);
        })
        .catch((err) => {
            console.error(err);
        });
};

const downloadEmail = () => {
    const blob = new Blob([emailContent.value], { type: "message/rfc822" });
    const url = window.URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url;
    link.setAttribute("download", "email.eml");
    document.body.appendChild(link);
    link.click();
    window.URL.revokeObjectURL(url);
};
</script>
<style scoped>
.container {
    background-image: url("https://img.freepik.com/premium-photo/file-folders-with-documents-background_488220-551.jpg");
    background-repeat: no-repeat;
    background-size: cover;
    max-width: 600px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 5px;
    text-align: center;
    justify-content: center;
    width: 1000px;
    height: 100vh;
}

input[type="file"] {
    margin-bottom: 10px;
}

.file12 {
    justify-content: center;
    margin-top: 40px;
    margin-bottom: 40px;
    margin-left: 60px;
}

button {
    margin: 0 5px;
    padding: 10px 20px;
    font-size: 16px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.red3 {
    display: flex;
    flex-direction: row;
    justify-content: center;
}

button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
}

.preview {
    margin-top: 20px;
    text-align: left;
}

.preview pre {
    background-color: #f9f9f9;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    white-space: pre-wrap;
    word-wrap: break-word;
}
</style>
<template>
    <div class="col-lg-8 mx-auto p-4 py-md-5">
        <main>
            <div class="container">
                <header class="d-flex align-items-center pb-3 mb-5 border-bottom" style="font-weight:40px;">
                    <span class="fs-4">Email Preview and Download</span>
                </header>
                <div class="file12">
                    <input type="file" id="fileInput" @change="handleFileChange" accept=".eml" />
                </div>
                <div class="red3">
                    <button @click="previewEmail" :disabled="!emailContent">Preview</button>
                    <button @click="downloadEmail" :disabled="!emailContent">Download</button>
                </div>
                <div v-if="previewVisible" class="mt-3" id="email-container">
                    <div id="subject-content" class="container-label">{{ emailInfo.subject }}</div>
                    <div class="email-info-row">
                        <div class="email-info-label">From</div>
                        <div class="content">{{ emailInfo.from }}</div>
                    </div>
                    <div class="email-info-row">
                        <div class="email-info-label">To</div>
                        <div class="content">{{ emailInfo.to }}</div>
                    </div>
                    <div class="email-info-row">
                        <div class="email-info-label">Cc</div>
                        <div class="content">{{ emailInfo.cc }}</div>
                    </div>
                    <div class="email-info-row">
                        <div class="email-info-label">Bcc</div>
                        <div class="content">{{ emailInfo.bcc }}</div>
                    </div>
                    <div class="email-info-row">
                        <div class="email-info-label">Date</div>
                        <div class="content">{{ emailInfo.date }}</div>
                    </div>
                    <div id="html-container">
                        <div class="container-label">HTML</div>
                        <div id="html-content">{{ emailInfo.html }}</div>
                    </div>
                    <div id="text-container">
                        <div class="container-label">Text</div>
                        <div id="text-content">{{ emailInfo.text }}</div>
                    </div>
                    <div id="attachments-container">
                        <div class="container-label">Attachments</div>
                        <div class="content">
                            <a v-for="(attachment, index) in emailInfo.attachments" :key="index" :href="attachment.url"
                                class="attachment-link">{{ attachment.name }}</a>
                        </div>
                    </div>
                </div>
            </div>
        </main>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import emlformat from 'eml-format';

const emailContent = ref('');
const previewVisible = ref(false);
const emailInfo = ref({
    subject: '',
    from: '',
    to: '',
    cc: '',
    bcc: '',
    date: '',
    html: '',
    text: '',
    attachments: []
});

const handleFileChange = (event) => {
    const file = event.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = () => {
        emailContent.value = reader.result;
        parseEmail(reader.result);
    };
    reader.readAsText(file);
};

const parseEmail = (eml) => {
    emlformat.read(eml, function (error, data) {
        if (error) {
            console.error(error);
            return;
        }
        emailInfo.value = {
            subject: data.subject,
            from: data.from,
            to: data.to,
            cc: data.cc,
            bcc: data.bcc,
            date: data.date,
            html: data.html,
            text: data.text,
            attachments: data.attachments || []
        };
    });
};

const previewEmail = () => {
    previewVisible.value = true;
};

const downloadEmail = () => {
    const blob = new Blob([emailContent.value], { type: 'text/plain' });
    const url = window.URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.setAttribute('download', 'email.eml');
    document.body.appendChild(link);
    link.click();
    window.URL.revokeObjectURL(url);
};
</script>

<style scoped>
.container {
    background-image: url('https://img.freepik.com/premium-photo/file-folders-with-documents-background_488220-551.jpg');
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

#html-content iframe {
    width: 1px;
    min-width: 100%;
    padding: 0;
    margin: 0;
    border: 0;
}

#html-content,
#text-content {
    border: 1px solid #ccc;
}

#text-content {
    font-family: Sans-serif;
    white-space: pre-wrap;
    font-family: Sans-serif;
    font-size: 13px;
    padding: 10px;
}

.email-info-row {
    display: flex;
}

.email-info-label {
    margin-right: 5px;
}

#attachments-container .content {
    display: flex;
}

#html-container,
#text-container,
#attachments-container {
    margin-top: 20px;
    padding-top: 20px;
    border-top: 1px solid #aaa;
}

.container-label {
    font-weight: 900;
    padding-bottom: 5px;
}

.attachment-link {
    display: block;
    margin-right: 5px;
    padding: 5px;
    border: 1px solid #aaa;
    border-radius: 2px;
}

#subject-content {
    font-weight: 900;
    font-size: 1.2rem;
}

.email-info-row {
    padding-top: 5px;
}
</style>
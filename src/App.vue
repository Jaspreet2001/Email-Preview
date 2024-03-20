<template>
  <div>

    <div class="fs-4">Email Preview and Download</div>

    <div class="text-center">
      <div class="file12">
        <input type="file" id="fileInput" @change="handleFileChange" accept=".eml" />
      </div>
    </div>
    <div class="red3 text-center">
      <button @click="previewEmail" :disabled="!emailInfo.subject">Preview</button>
      <button @click="downloadEmail" :disabled="!emailContent">Download</button>
    </div>
    <div v-if="previewVisible" class="mt-3" id="email-container">
      <div id="subject-content" class="container-label">{{ emailInfo.subject }}</div>
      <div class="email-info-row">
        <div class="email-info-label">From : </div>
        <div class="content">{{ emailInfo.from.name }} &lt;{{ emailInfo.from.address }}&gt;</div>
      </div>
      <div class="email-info-row">
        <div class="email-info-label">To : </div>
        <div class="content">{{ emailInfo.to[0].name }} &lt;{{ emailInfo.to[0].address }}&gt;</div>
      </div>
      <div class="email-info-row" v-if="emailInfo.date">
        <div class="email-info-label">Date : </div>
        <div class="content">{{ formatDate(emailInfo.date) }}</div>
      </div>
      <div id="html-container">
        <div class="container-label">HTML </div>
        <div id="html-content" v-html="emailInfo.html"></div>
        <div>
          <small><em>Images not shown? Mixed content is not allowed, so check that image links are not HTTP if this
              page is HTTPS.</em></small>
        </div>
      </div>
      <div id="text-container" v-if="emailInfo.text">
        <div class="container-label">Text </div>
        <div id="text-content">{{ emailInfo.text }}</div>
      </div>
      <div id="attachments-container" v-if="emailInfo.attachments && emailInfo.attachments.length">
        <div class="container-label">Attachments</div>
        <div class="content">
          <button v-for="(attachment, index) in emailInfo.attachments" :key="index"
            @click="downloadAttachment(attachment)" class="attachment-link">
            {{ attachment.filename }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import PostalMime from '../node_modules/postal-mime/src/postal-mime.js';

const previewVisible = ref(false);
const emailContent = ref('');
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
  const parser = new PostalMime();
  parser.parse(eml)
    .then(email => {
      emailInfo.value = {
        subject: email.subject || '',
        from: email.from || '',
        to: email.to || '',
        cc: email.cc || '',
        bcc: email.bcc || '',
        date: email.date || '',
        html: email.html || '',
        text: email.text || '',
        attachments: email.attachments || []
      };
      previewVisible.value = true;
    })
    .catch(error => {
      console.error(error);
    });
};

const formatDate = (date) => {
  if (!date) return '';
  return new Date(date).toLocaleString();
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
const downloadAttachment = (attachment) => {
  const blob = new Blob([attachment.content], { type: attachment.mimeType });
  const url = window.URL.createObjectURL(blob);
  const link = document.createElement('a');
  link.href = url;
  link.setAttribute('download', attachment.filename || 'attachment');
  document.body.appendChild(link);
  link.click();
  window.URL.revokeObjectURL(url);
};

</script>
<style scoped>
.fs-4 {
  font-style: normal;
  padding: 20px;
  font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
  color: rgb(178, 12, 12);
  text-shadow: #aaa;
  background-color: rgb(248, 250, 227);
  font-size: 40px;
  text-align: center;
  margin: 40px;
}

.mt-3 {
  margin: 50px;
  padding: 50px;
}


.text-center {
  text-align: center;
}

.file12 {
  display: inline-block;
}

.red3 {
  display: inline-block;
}

.attachment-link {
  display: block;
  margin: 5px 0;
}

.container {

  max-width: 2500px;
  height: 100%;
  margin: 0 auto;
  padding: 20px;
  background-color: rgb(240, 244, 223);
  background-repeat: no-repeat;
  background-size: cover;
  border: 1px solid #ccc;
  border-radius: 5px;
  text-align: center;
  justify-content: center;
  width: 1000px;
  height: 100vh;
}

* {
  margin: 20;
  padding: 20;
  font-family: 'montserrat', sans-serif;
  height: 100%;

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
  background-color: hsl(263, 100%, 50%);
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
  padding-top: 20px;
  text-align: left;
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

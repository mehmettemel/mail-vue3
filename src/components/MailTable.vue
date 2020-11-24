<template>
  <table class="mail-table">
    <tbody>
      <tr
        v-for="email in unarchivedEmails"
        :key="email.id"
        :class="['clickable', email.read ? 'read' : '']"
        @click="openEmail(email)"
      >
        <td><input type="checkbox" /></td>
        <td>{{ email.from }}</td>
        <td>
          <p>
            <strong>{{ email.subject }}</strong>
            -
            {{ email.body }}
          </p>
        </td>
        <td class="date">{{ format(new Date(email.sentAt), "MMM do yyyy") }}</td>
        <td><button @click="archiveEmail(email)">Archive</button></td>
      </tr>
    </tbody>
  </table>
  <MailView v-if="openedEmail" :email="openedEmail" />
  <div v-if="openedEmail">
    {{ openedEmail.subject }}
  </div>
</template>

<script>
import MailView from "./MailView";
import { format } from "date-fns";
import axios from "axios";
import { ref } from "vue";
export default {
  async setup() {
    let response = await axios.get("http://localhost:3000/emails");
    let emails = response.data;
    // await new Promise(resolve => setTimeout(resolve, 900));
    return {
      format,
      emails: ref(emails),
      openedEmail: null
    };
  },
  components: {
    MailView
  },
  computed: {
    sortedEmails() {
      return this.emails.sort((e1, e2) => {
        return e1.sentAt < e2.sentAt ? 1 : -1;
      });
    },

    unarchivedEmails() {
      return this.sortedEmails.filter(e => !e.archived);
    }
  },
  methods: {
    openEmail(email) {
      email.read = true;
      this.uptadeEmail(email);
      this.openedEmail = email;
    },
    archiveEmail(email) {
      email.archived = true;
      this.uptadeEmail(email);
    },
    uptadeEmail(email) {
      axios.put(`http://localhost:3000/emails/${email.id}`, email);
    }
  }
};
</script>

<style></style>

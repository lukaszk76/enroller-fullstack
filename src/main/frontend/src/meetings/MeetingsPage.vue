<template>
  <div>
    <div :class="'alert alert-' + (this.isError ? 'error' : 'success')" v-if="message">{{ message }}</div>
    <new-meeting-form @added="addNewMeeting($event)"></new-meeting-form>

    <span v-if="meetings.length == 0">
               Brak zaplanowanych spotkań.
           </span>
    <h3 v-else>
      Zaplanowane zajęcia ({{ meetings.length }})
    </h3>

    <meetings-list :meetings="meetings"
                   :username="username"
                   @attend="addMeetingParticipant($event)"
                   @unattend="removeMeetingParticipant($event)"
                   @delete="deleteMeeting($event)"></meetings-list>
  </div>
</template>

<script>
    import NewMeetingForm from "./NewMeetingForm";
    import MeetingsList from "./MeetingsList";

    export default {
        components: {NewMeetingForm, MeetingsList},
        props: ['username'],
        data() {
            return {
                meetings: [],
                message: '',
                isError: false
            };
        },
        methods: {
            addNewMeeting(meeting) {
                
                this.clearMessage();
                this.$http.post('meetings', meeting)
                    .then(response => {
                        meeting.id = response.data;
                        this.meetings.push(meeting);
                        this.success('Spotkanie zostało dodane. Możesz się zapisać.');
                    })
                    .catch(response => this.failure('Błąd przy dodawaniu spotkania. Kod odpowiedzi: ' + response.status));
            },

            addMeetingParticipant(meeting) {
                var query = "meetings/" + meeting.id + "/" + this.username;
                this.$http.post(query)
                    .then( () => {
                      this.success('Pomyślnie zapisałeś sie na spotkanie.');
                      meeting.participants.push(this.username);
                    })
                    .catch(response => this.failure('Błąd przy zapisie uczestnika do spotkania. Kod odpowiedzi: ' + response.status));
            },

            removeMeetingParticipant(meeting) {
                var query = "meetings/" + meeting.id + "/" + this.username;
                this.$http.delete(query)
                    .then( () => {
                      this.success('Pomyślnie wypisałeś sie ze spotkania.');
                      meeting.participants.splice(meeting.participants.indexOf(this.username), 1);
                    })
                    .catch(response => this.failure('Błąd przy wypisywaniu uczestnika ze spotkania. Kod odpowiedzi: ' + response.status));
                
            },
            
            deleteMeeting(meeting) {
                this.meetings.splice(this.meetings.indexOf(meeting), 1);
            },

            success(message) {
                this.message = message;
                this.isError = false;
            },
            failure(message) {
                this.message = message;
                this.isError = true;
            },
            clearMessage() {
                this.message = undefined;
            }
        },

 //       created() {
   //         this.$http.get("meetings")
     //       .then(response => {
       //         this.meetings = JSON.parse(response.body);
                //for (meeting in this.meetings) {
                //    var query = "meetings/" + meeting.id + "/participants";
                //    this.$http.get(query)
                //    .then(response => {
                //        meeting["participants"] = JSON.parse(response.body);
                //    })
                //    .catch(() => this.failure('Nie udało sie pobrac listy uczestnikow dla spotkania ' + meeting.title ));
                //}
  //           })
    //        .catch(() => this.failure('Nie udało sie pobrać listy spotkań.' + response.status));
      //  },

    }
</script>
<style lang="scss">
 .alert {
    padding: 10px;
    margin-bottom: 10px;
    border: 2px solid black;
    &-success {
      background: lightgreen;
      border-color: darken(lightgreen, 10%);
    }
    &-error {
      background: indianred;
      border-color: darken(indianred, 10%);
      color: white;
    }
 }
</style>

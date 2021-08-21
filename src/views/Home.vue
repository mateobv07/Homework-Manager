<template>
  <v-row class="fill-height">
    <v-col>
      <v-row>
        <v-sheet
          height="64"
          :width="$vuetify.breakpoint.smAndDown ? '100%' : '57%'"
          :class="
            $vuetify.breakpoint.smAndDown
              ? 'mt-6 mx-4 pr-9'
              : 'ml-12 mt-10 .rounded-lg'
          "
        >
          <v-toolbar color="cyan lighten-4" flat class="rounded-t-lg">
            <v-btn
              outlined
              :class="$vuetify.breakpoint.smAndDown ? 'mx-auto' : ' mr-4'"
              color="grey darken-2"
              @click="setToday"
            >
              Today
            </v-btn>
            <v-btn fab text small color="grey darken-2" @click="prev">
              <v-icon small> mdi-chevron-left </v-icon>
            </v-btn>
            <v-btn fab text small color="grey darken-2" @click="next">
              <v-icon small> mdi-chevron-right </v-icon>
            </v-btn>
            <v-toolbar-title v-if="$refs.calendar">
              {{ $refs.calendar.title }}
            </v-toolbar-title>
            <v-spacer></v-spacer>
            <v-menu bottom right>
              <template v-slot:activator="{ on, attrs }">
                <v-btn outlined color="grey darken-2" v-bind="attrs" v-on="on">
                  <span>{{ typeToLabel[type] }}</span>
                  <v-icon right> mdi-menu-down </v-icon>
                </v-btn>
              </template>
              <v-list>
                <v-list-item @click="type = 'week'">
                  <v-list-item-title>Week</v-list-item-title>
                </v-list-item>
                <v-list-item @click="type = 'month'">
                  <v-list-item-title>Month</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-menu>
          </v-toolbar>
        </v-sheet>
        <v-sheet
          :height="$vuetify.breakpoint.smAndDown ? '550' : '600'"
          :width="$vuetify.breakpoint.smAndDown ? '100%' : '57%'"
          :class="$vuetify.breakpoint.smAndDown ? 'mx-5' : 'ml-12'"
        >
          <v-calendar
            ref="calendar"
            v-model="focus"
            color="primary"
            :events="events"
            :event-color="getEventColor"
            :type="type"
            @click:event="showEvent"
            :event-more="false"
            @change="updateRange"
          ></v-calendar>

          <v-menu
            v-model="selectedOpen"
            :close-on-content-click="false"
            :activator="selectedElement"
            offset-x
          >
            <v-card color="grey lighten-4" min-width="350px">
              <v-toolbar :color="selectedEvent.color" dark>
                <v-btn icon>
                  <v-icon @click="view_tarea(), (selectedOpen = false)"
                    >mdi-pencil</v-icon
                  >
                </v-btn>
                <v-toolbar-title v-html="selectedEvent.name"></v-toolbar-title>
                <v-spacer></v-spacer>
              </v-toolbar>
              <v-card-text>
                <p v-html="selectedEvent.details"></p>
              </v-card-text>
              <v-card-actions>
                <v-btn text color="secondary" @click="selectedOpen = false">
                  Cancel
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-menu>
        </v-sheet>

        <v-card
          elevation="4"
          :class="
            $vuetify.breakpoint.smAndDown
              ? 'mx-auto rounded-lg justify-center mt-5'
              : 'mx-auto rounded-lg justify-center'
          "
          :width="$vuetify.breakpoint.smAndDown ? '90%' : '34%'"
        >
          <v-card-title class="blue lighten-4 justify-center">
            <span class="text-h3 text-center"> Tareas </span>
            <v-btn
              absolute
              right
              top
              color="white"
              class="text--primary mt-10"
              fab
              @click="
                (tarea_date = null),
                  (tarea_name = ''),
                  (tarea_description = ''),
                  (new_tarea = true)
              "
            >
              <v-icon>mdi-plus</v-icon>
            </v-btn>
          </v-card-title>

          <v-divider></v-divider>
          <v-card-actions>
            <v-virtual-scroll
              :items="events"
              :item-height="50"
              :height="$vuetify.breakpoint.smAndDown ? '320' : '450'"
              max-width="550"
              class="mx-auto pb-8"
            >
              <template v-slot:default="{ item }">
                <v-list-item @click="(selectedEvent = item), view_tarea()">
                  <v-list-item-avatar>
                    <v-avatar
                      :color="item.color"
                      size="56"
                      class="white--text pr-4"
                    >
                      {{ events.indexOf(item) + 1 }}
                    </v-avatar>
                  </v-list-item-avatar>

                  <v-list-item-content>
                    <v-list-item-title>{{ item.name }}</v-list-item-title>
                  </v-list-item-content>

                  <v-btn depressed small>
                    {{ item.start.toString().substring(0, 15) }}
                  </v-btn>
                </v-list-item>
              </template>
            </v-virtual-scroll>
          </v-card-actions>
          <v-dialog v-model="selectedOpen_tarea" max-width="600">
            <v-card color="grey lighten-4">
              <v-toolbar :color="selectedEvent.color" dark>
                <v-toolbar-title> {{ tarea_name }}</v-toolbar-title>
                <v-spacer></v-spacer>
                <v-btn icon>
                  <v-icon @click="delete_dialog = true" >mdi-delete</v-icon>
                </v-btn>
              </v-toolbar>
              <v-col md="10" class="mx-auto">
                <v-text-field
                  v-model="tarea_name"
                  label="Name"
                  solo
                ></v-text-field>

                <v-menu
                  v-model="menu2"
                  :close-on-content-click="false"
                  :nudge-right="40"
                  transition="scale-transition"
                  offset-y
                  min-width="auto"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="computedDateFormatted"
                      label="Fecha de entrega"
                      prepend-icon="mdi-calendar"
                      readonly
                      v-bind="attrs"
                      v-on="on"
                    ></v-text-field>
                  </template>
                  <v-date-picker
                    v-model="tarea_date"
                    @input="menu2 = false"
                  ></v-date-picker>
                </v-menu>
                <v-textarea
                  v-model="tarea_description"
                  :color="selectedEvent.color"
                  label="Decription"
                  required
                  outlined
                ></v-textarea>
              </v-col>
              <v-card-actions class="justify-center mt-n5">
                <v-btn class="secondary" @click="selectedOpen_tarea = false">
                  Cancel
                </v-btn>
                <v-btn color="success" @click="update_tarea(selectedEvent.id)"> Save </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-card>
        <v-dialog v-model="new_tarea" max-width="600">
          <v-card color="grey lighten-4">
            <v-toolbar color="blue-grey darken-2" dark>
              <v-toolbar-title> Nueva Tarea</v-toolbar-title>
              <v-spacer></v-spacer>
              <v-btn icon>
                <v-icon @click="new_tarea = false">mdi-close</v-icon>
              </v-btn>
            </v-toolbar>
            <v-col md="10" class="mx-auto">
              <v-text-field
                v-model="tarea_name"
                label="Name"
                solo
              ></v-text-field>

              <v-menu
                v-model="menu1"
                :close-on-content-click="false"
                :nudge-right="40"
                transition="scale-transition"
                offset-y
                min-width="auto"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                    v-model="computedDateFormatted"
                    label="Fecha de entrega"
                    prepend-icon="mdi-calendar"
                    readonly
                    v-bind="attrs"
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="tarea_date"
                  @input="menu1 = false"
                ></v-date-picker>
              </v-menu>
              <v-textarea
                v-model="tarea_description"
                :color="selectedEvent.color"
                label="Decription"
                required
                outlined
              ></v-textarea>
            </v-col>
            <v-card-actions class="justify-center mt-n5 pb-4">
              <v-btn color="success" @click="create_tarea()"> Agregar </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="delete_dialog" max-width="300">
            <v-card color="grey lighten-4">
              <v-toolbar color="error" dark class="justify-content text-center">
              <v-spacer></v-spacer>
                <v-toolbar-title > DELETE ?</v-toolbar-title>
                <v-spacer></v-spacer>
              </v-toolbar>
              <v-card-actions class="justify-center mt-5">
                <v-btn class="secondary" @click="delete_dialog = false">
                  Cancel
                </v-btn>
                <v-btn color="error" @click="delete_tarea(selectedEvent.id)"> Delete </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
      </v-row>
    </v-col>
  </v-row>
</template>

<script>
/* eslint-disable */
import axios from "axios";
export default {
  name: "Home",

  components: {},
  data: () => ({
    focus: "",
    type: "month",
    typeToLabel: {
      month: "Month",
      week: "Week",
    },
    selectedEvent: {},
    selectedElement: null,
    selectedOpen: false,
    new_tarea: false,
    delete_dialog: false,
    tarea_name: "",
    tarea_date: null,
    tarea_dateFormatted: "",
    tarea_description: "",
    selectedOpen_tarea: false,
    menu2: false,
    menu1: false,
    events: [],
    colors: ["blue", "indigo", "deep-purple", "cyan", "green", "orange"],
    names: [
      ["Meeting", "August 18, 2021", "helooo", "321"],
      ["Holiday", "August 19, 2021", "helooo", "123"],
      ["PTO", "August 27, 2021", "helooo", "45"],
      ["Travel", "August 11, 2021", "helooo", "77"],
      ["Event", "August 5, 2021", "helooo", "99"],
      ["Birthday", "August 29, 2021", "helooo", '43'],
      ["Birthday", "May 29, 2021", "helooo", '44'],
      ["TEST", "2021-8-15", "helooo", '87'],
    ],
  }),
  computed: {
    computedDateFormatted() {
      return this.formatDate(this.tarea_date);
    },
  },
  mounted() {
    this.get_all_homework();
    this.$refs.calendar.checkChange();
  },
  methods: {
    get_all_homework() {
      axios({
        method: "get",
        url: "http://127.0.0.1:8000/homeworks/",
        auth: {
          username: "admin",
          password: "Thebahamas1",
        },
      })
        .then(function (response) {
          console.log(response);
          for (let i = 0; i < response.data.length; i++) {
           names.push([
             response.data[i].title, response.data[i].date, response.data[i].description, response.data[i].id
           ])
          }
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    create_tarea() {
      var vueinstance = this
      axios({
        method: "post",
        url: "http://127.0.0.1:8000/homeworks/",
        data: {
          title: this.tarea_name,
          date: this.tarea_date,
          description: this.tarea_description,
        },
        auth: {
          username: "admin",
          password: "Thebahamas1",
        },
      })
        .then(function (response) {
          console.log("worked");
          vueinstance.new_tarea = false
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    formatDate(date) {
      console.log(date);
      if (!date) return null;
      const [year, month, day] = date.split("-");
      var check = `${month}/${day}/${year}`;
      const dia = new Date(check);
      return dia.toString().substring(0, 15);
    },
    update_tarea(id) {
      console.log(id)
      var vueinstance = this
      axios({
        method: "patch",
        url: "http://127.0.0.1:8000/homeworks/" + id + "/",
        data: {
          title: this.tarea_name,
          date: this.tarea_date,
          description: this.tarea_description,
        },
        auth: {
          username: "admin",
          password: "Thebahamas1",
        },
      })
        .then(function (response) {
          console.log("worked");
          vueinstance.selectedOpen_tarea = false;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    delete_tarea(id) {
      console.log(id)
      var vueinstance = this
      axios({
        method: "delete",
        url: "http://127.0.0.1:8000/homeworks/" + id + "/",
        auth: {
          username: "admin",
          password: "Thebahamas1",
        },
      })
        .then(function (response) {
          vueinstance.delete_dialog = false;
          vueinstance.selectedOpen_tarea = false;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    view_tarea() {
      this.tarea_name = this.selectedEvent.name;
      this.tarea_description = this.selectedEvent.details;
      var day = this.selectedEvent.start.getDate().toString();
      var month = this.selectedEvent.start.getMonth() + 1;
      month = month.toString();
      var year = this.selectedEvent.start.getFullYear().toString();
      var formatdate = `${year}-${month}-${day}`;
      this.tarea_date = formatdate;
      this.selectedOpen_tarea = true;
    },
    getEventColor(event) {
      return event.color;
    },
    setToday() {
      this.focus = "";
    },
    prev() {
      this.$refs.calendar.prev();
    },
    next() {
      this.$refs.calendar.next();
    },
    showEvent({ nativeEvent, event }) {
      const open = () => {
        this.selectedEvent = event;
        this.selectedElement = nativeEvent.target;
        requestAnimationFrame(() =>
          requestAnimationFrame(() => (this.selectedOpen = true))
        );
      };

      if (this.selectedOpen) {
        this.selectedOpen = false;
        requestAnimationFrame(() => requestAnimationFrame(() => open()));
      } else {
        open();
      }

      nativeEvent.stopPropagation();
    },
    updateRange({ start, end }) {
      //sort by date
      const events = [];
      var lol = this.names.sort(function (a, b) {
        var dateA = new Date(a[1]),
          dateB = new Date(b[1]);
        return dateA - dateB;
      });
      this.names = lol;
      const eventCount = this.names.length;

      for (let i = 0; i < eventCount; i++) {
        const firstTimestamp = new Date(this.names[i][1]);
        const dia_entrega = new Date(
          firstTimestamp - (firstTimestamp % 900000)
        );

        events.push({
          name: this.names[i][0],
          start: dia_entrega,
          color: this.colors[this.rnd(0, this.colors.length - 1)],
          details: this.names[i][2],
          id: this.names[i][3],
        });
      }

      this.events = events;
    },
    rnd(a, b) {
      return Math.floor((b - a + 1) * Math.random()) + a;
    },
  },
};
</script>

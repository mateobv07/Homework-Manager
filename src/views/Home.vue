<template>
  <v-row class="fill-height">
    <v-col>
      <v-row>
        <v-sheet height="64" :width="$vuetify.breakpoint.smAndDown ? '100%' : '57%'"  :class="$vuetify.breakpoint.smAndDown ? 'mt-6 mx-4 pr-9': 'ml-12 mt-10 .rounded-lg'">
          <v-toolbar color="cyan lighten-4" flat class="rounded-t-lg">
            <v-btn
              outlined
              :class="$vuetify.breakpoint.smAndDown ? 'mx-auto' :' mr-4'"
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
        <v-sheet :height="$vuetify.breakpoint.smAndDown ? '550' : '600'" :width="$vuetify.breakpoint.smAndDown ? '100%' : '57%'" :class="$vuetify.breakpoint.smAndDown ? 'mx-5' : 'ml-12'">
          <v-calendar
            ref="calendar"
            v-model="focus"
            color="primary"
            :events="events"
            :event-color="getEventColor"
            :type="type"
            @click:event="showEvent"
           :event-more="false"
            @click:date="viewDay"
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
                  <v-icon
                    @click="(selectedOpen_tarea = true), (selectedOpen = false)"
                    >mdi-pencil</v-icon
                  >
                </v-btn>
                <v-toolbar-title v-html="selectedEvent.name"></v-toolbar-title>
                <v-spacer></v-spacer>
                <v-btn icon>
                  <v-icon>mdi-dots-vertical</v-icon>
                </v-btn>
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
          :class="$vuetify.breakpoint.smAndDown ? 'mx-auto rounded-lg justify-center mt-5' : 'mx-auto rounded-lg justify-center'"
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
            >
              <v-icon>mdi-plus</v-icon>
            </v-btn>
          </v-card-title>

          <v-divider></v-divider>
          <v-card-actions>
            <v-virtual-scroll
              :items="events"
              :item-height="50"
              height="450"
              max-width="550"
              class="mx-auto"
            >
              <template v-slot:default="{ item }">
                <v-list-item
                  @click="(selectedEvent = item), (selectedOpen_tarea = true)"
                >
                  <v-list-item-avatar>
                    <v-avatar :color="item.color" size="56" class="white--text pr-4"> {{events.indexOf(item) + 1}}
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
          <v-dialog v-model="selectedOpen_tarea" width="600">
            <v-card color="grey lighten-4" min-width="350px">
              <v-toolbar :color="selectedEvent.color" dark>
                <v-btn icon>
                  <v-icon>mdi-pencil</v-icon>
                </v-btn>
                <v-toolbar-title> {{ selectedEvent.name }}</v-toolbar-title>
                <v-spacer></v-spacer>
                <v-btn icon>
                  <v-icon>mdi-dots-vertical</v-icon>
                </v-btn>
              </v-toolbar>
              <v-card-text>
                <p class="mt-5">{{ selectedEvent.details }}</p>
              </v-card-text>
              <v-card-actions>
                <v-btn
                  text
                  color="secondary"
                  @click="selectedOpen_tarea = false"
                >
                  Cancel
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-card>
      </v-row>
    </v-col>
  </v-row>
</template>

<script>
/* eslint-disable */
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
    selectedElement_tarea: null,
    selectedOpen_tarea: false,
    events: [],
    colors: ["blue", "indigo", "deep-purple", "cyan", "green", "orange"],
    names: [
      ["Meeting", "August 17, 2021"],
      ["Holiday", "August 19, 2021"],
      ["PTO", "August 17, 2021"],
      ["Travel", "August 17, 2021"],
      ["Event", "August 17, 2021"],
      ["Birthday", "August 25, 2021"],
    ],
  }),
  mounted() {
    this.$refs.calendar.checkChange();
  },
  methods: {
    viewDay({ date }) {
      this.focus = date;
      // que salgan las fechas en el v-sheet de a lado
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
      const events = [];

      const eventCount = this.names.length;

      for (let i = 0; i < eventCount; i++) {
        const firstTimestamp = new Date(this.names[i][1]);
        const dia_entrega = new Date(
          firstTimestamp - (firstTimestamp % 900000)
        );
        console.log("first");
        console.log(firstTimestamp);
        events.push({
          name: this.names[i][0],
          start: dia_entrega,
          color: this.colors[this.rnd(0, this.colors.length - 1)],
          details: "helloooooo",
          // timed: !allDay,
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

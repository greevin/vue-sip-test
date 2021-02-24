<template>
  <div>
    <b-navbar type="dark" variant="dark">
      <b-navbar-brand href="#">Vue JsSIP</b-navbar-brand>
      <b-navbar-nav class="ml-auto">
        <!-- Navbar dropdowns -->
        <b-dropdown id="dropdown-1" right no-caret>
          <template #button-content>
            <i class="fa fa-phone"></i>
          </template>
          <div class="pt-4 notification-body" id="listCall">
            <strong>Ligações</strong>
            <hr />
            <div v-for="(item, index) in notifications" :key="index">
              <b-row>
                <b-col cols="10" class="d-flex flex-column"
                  ><span>{{ item.name }}</span>
                  <small>{{ item.sip }}</small></b-col
                >
                <b-col
                  cols="2"
                  class="d-flex justify-content-center align-items-center"
                  style="cursor: pointer"
                >
                  <i class="fa fa-phone" @click="clickCall(item.sip)"></i>
                </b-col>
              </b-row>
              <hr />
            </div>
          </div>
        </b-dropdown>
      </b-navbar-nav>
    </b-navbar>
  </div>
</template>

<script>
import notifications from "../data/notifications";
export default {
  name: "TopNavBar",
  data() {
    return {
      notifications,
      dest: "",
    };
  },
  methods: {
    clickCall(sip) {
      this.dest = sip;
      console.log(sip);
    },
  },
};
</script>

<style scoped>
.fa-phone {
  font-size: 24px;
}
</style>
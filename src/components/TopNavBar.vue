<template>
  <div>
    <b-navbar type="dark" variant="dark">
      <b-navbar-brand href="#">Vue JsSIP</b-navbar-brand>
      <b-navbar-nav class="ml-auto">
        <!-- Navbar dropdowns -->
        <b-dropdown id="dropdown-1" right>
          <template #button-content>
            <i class="fa fa-phone"></i>
          </template>
          <div class="p-2 pt-0 notification-body">
            <strong>Ligações</strong>
            <b-dropdown-divider></b-dropdown-divider>
            <b-dropdown-item
              v-for="(item, index) in notifications"
              :key="index"
            >
              <b-container class="bv-example-row">
                <b-row>
                  <b-col cols="10"
                    ><span>{{ item.name }}</span>
                    <br />
                    <small>{{ item.sip }}</small></b-col
                  >
                  <b-col cols="2"><i class="fa fa-phone"></i></b-col>
                </b-row>
              </b-container>
            </b-dropdown-item>
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
    };
  },
};
</script>

<style>
</style>
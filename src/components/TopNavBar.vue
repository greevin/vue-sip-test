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
          <div class="p-2 pt-0 notification-body">
            <strong>Ligações</strong>
            <b-dropdown-divider></b-dropdown-divider>
            <b-dropdown-item
              v-for="(item, index) in notifications"
              :key="index"
            >
              <b-row>
                <b-col
                  cols="10"
                  style="padding-left: 0px"
                  class="d-flex flex-column"
                  ><span>{{ item.name }}</span>
                  <small>{{ item.sip }}</small></b-col
                >
                <b-col
                  cols="2"
                  style="padding-right: 0px"
                  class="d-flex justify-content-center align-items-center"
                  ><i class="fa fa-phone"></i
                ></b-col>
              </b-row>
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

<style scoped>
.fa-phone {
  font-size: 24px;
}
</style>
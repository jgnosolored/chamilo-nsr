<template>
  <div v-if="item && item['resourceLinkListFromEntity']">
    <v-card>
      <v-list-item
          v-for="link in item['resourceLinkListFromEntity']"
      >
        <v-list-item-content>
          <div v-if="link['course']">
            <v-icon icon="mdi-book"/>
            {{ $t('Course') }}: {{ link.course.resourceNode.title }}
          </div>

          <div v-if="link['session']">
            <v-icon icon="mdi-book-open"/>
            {{ $t('Session') }}: {{ link.session.name }}
          </div>

          <div v-if="link['group']">
            <v-icon icon="mdi-people"/>
            {{ $t('Group') }}: {{ link.group.resourceNode.title }}
          </div>

          <div v-if="link['userGroup']">
            {{ $t('Class') }}: {{ link.userGroup.resourceNode.title }}
          </div>

          <div v-if="link['user']">
            <v-icon icon="mdi-account"/>
            {{ $t('User') }}:
            <!--          <q-avatar size="32px">-->
            <!--            <img :src="link.user.illustrationUrl + '?w=80&h=80&fit=crop'" />-->
            <!--          </q-avatar>-->
            {{ link.user.username }}
          </div>

          <q-select
              filled
              v-model="link.visibility"
              :options="visibilityList"
              label="Status"
              emit-value
              map-options
              v-if="showStatus"
          />
        </v-list-item-content>
      </v-list-item>
    </v-card>
  </div>

  <VueMultiselect
      placeholder="Share with User"
      v-model="selectedUsers"
      :loading="isLoading"
      :options="users"
      :multiple="true"
      :searchable="true"
      :internal-search="false"
      @search-change="asyncFind"
      @select="addUser"
      limit-text="3"
      limit="3"
      label="username"
      track-by="id"
  />

</template>

<style src="vue-multiselect/dist/vue-multiselect.css"></style>

<script>

import {computed, ref, toRefs} from "vue";
import axios from "axios";
import {ENTRYPOINT} from "../../config/entrypoint";
import useVuelidate from "@vuelidate/core";
import VueMultiselect from 'vue-multiselect'
import isEmpty from 'lodash/isEmpty';
export default {
  name: 'EditLinks',
  components: {
    VueMultiselect,
  },
  props: {
    item: {
      type: Object,
      required: true
    },
    showStatus: {
      type: Boolean,
      required: false,
      default: true
    }
  },
  setup (props) {
    const visibilityList = [
      {value: 2, label: 'Published'},
      {value: 0, label: 'Draft'},
    ];

    const users = ref([]);
    const selectedUsers = ref([]);
    const isLoading = ref(false);

    function addUser(userResult) {
      if (isEmpty(props.item.resourceLinkListFromEntity)) {
        props.item.resourceLinkListFromEntity = [];
      }

      props.item.resourceLinkListFromEntity.push(
          {
            uid: userResult.id,
            user: { username: userResult.username},
            visibility: 2
          }
      );
    }

    function asyncFind(query) {
      if (query.toString().length < 3) {
        return;
      }

      isLoading.value = true;
      axios.get(ENTRYPOINT + 'users', {
        params: {
          username: query
        }
      }).then(response => {
        isLoading.value = false;
        let data = response.data;
        users.value = data['hydra:member'];
      }).catch(function (error) {
        isLoading.value = false;
        console.log(error);
      });
    }

    return {v$: useVuelidate(), visibilityList, users, selectedUsers, asyncFind, addUser, isLoading};
  },
};
</script>

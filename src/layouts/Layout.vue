<template>
  <q-layout view="hHh LpR fFf">
    <q-header  elevated v-if="$route.fullPath!=='/'">
      <q-toolbar class=" text-black constrain flex items-center">
        <div >
            <q-img @click="$router.push('/map')"  src="logo/logo_small.png" style="width:40px"></q-img>
        </div>

        <p class="text-h6 q-ml-md q-mb-none"> Cycle Planet</p>
        <q-space ></q-space> 
      
          <q-tabs  v-model="tab" shrink v-if="screenwidthbig">
            <q-btn flat name="Map" label="Map" to="/map"/>
            <q-btn v-if="teamMember" flat name="Admin" label="Admin panel" to="/admin"/>
            <q-btn flat name="Trips" label="Trips" to="/trips" />
            <q-btn flat name="Chat" label="Chat" :to="Object.keys(this.userMessagesSortedByDate).length?'/chat/'+Object.keys(this.userMessagesSortedByDate)[0]:'/chat'" v-if="loggedIn">
              <q-badge v-if="Object.keys(unreadchatlistnew).length" color="red">{{Object.keys(unreadchatlistnew).length}}</q-badge>
            </q-btn>
             <q-btn-dropdown  stretch flat label="More">
               <q-list>
                 <div v-for="(item, index) in sections" :key="index">
                 <q-item clickable v-close-popup :to="item.to" v-if="item.apple?!isApple:true">
                   <q-item-section avatar >
                    <q-avatar :icon="item.icon" />
                  </q-item-section>
                  <q-item-section>
                    <q-item-label>{{item.title}}</q-item-label>
                  </q-item-section>
                 </q-item>
                 </div>
               </q-list>
             </q-btn-dropdown>

          </q-tabs>

          <q-btn v-if="!loggedIn" dense flat to="/auth" label="Login" icon-right="account_circle"/>
          
          <q-btn class="q-mx-md" v-if="loggedIn" dense flat round  :to="'/user/'+myUserId">
            <q-avatar v-if="myUserDetails">
                  <img :src="myUserDetails.imageurl">
                </q-avatar>
          <!-- <q-menu>
            <div class="">
              <div class="">
                <q-list style="min-width: 100px">
                  <q-item clickable v-close-popup :to="'/user/'+myUserId">
                    <q-item-section>View profile</q-item-section>
                  </q-item>
                  <q-item clickable v-close-popup @click="myMarkersDialog=true">
                    <q-item-section>My markers</q-item-section>
                  </q-item>
                  <q-item clickable v-close-popup @click="checkMarkersDialog=true">
                    <q-item-section>Check markers</q-item-section>
                  </q-item>
                  <q-item clickable v-close-popup>
                    <q-item-section>
                      <q-btn dense :style="buttonStyle" label="Log out" @click="logoutUserMethod" to="/auth"/>
                    </q-item-section>
                  </q-item>
                </q-list>
              </div>
            </div>
          </q-menu> -->
          </q-btn>

          <q-btn dense flat round icon="menu" @click="rightDrawerOpen= !rightDrawerOpen" />
      </q-toolbar>
      <q-banner inline-actions dense rounded class="bg-secondary text-center cursor-pointer q-pa-xs" v-if="$route.fullPath==='/map'">
      <div class="no-padding" :style="isWebApp?'':'font-size:11px'" label="Learn more about Cycle Planet" @click="$router.push('/')" >Learn more about Cycle Planet</div>
      </q-banner>
    </q-header>
     
    <q-page-sticky v-if="isWebApp" position="right" :offset="[0, 0]" style="z-index:99999" class="justify-end">
      <q-btn @click="feedbackDialog=true" color="primary" label="feedback" class="rotate-270 text-black" style="margin-right:-40px"/>
    </q-page-sticky>

    <q-drawer v-model="rightDrawerOpen" side="right" overlay bordered content-class="bg-grey-2" :width="200">
      <drawer-right/>
    </q-drawer>

    
    <q-page-container :class="($route.fullPath==='/'||$route.fullPath==='/map')?'':'constrain'">
       <router-view />
    </q-page-container>

    <q-footer  class="layout" v-if="!$route.fullPath.includes('/chat/')">
      <q-tabs indicator-color="transparent" active-:style="buttonStyle"  class="bg-white text-grey shadow-2" align="justify" :breakpoint="0">
          <q-tab name="map" icon="map"   @click="$router.push('/map')" />
          <q-tab name="trips" icon="alt_route"  @click="$router.push('/trips')"/>
          <q-tab v-if="loggedIn" icon="chat" @click="$router.push('/chat')">
            <q-badge color="red" v-if="Object.keys(unreadchatlistnew).length" floating>{{Object.keys(unreadchatlistnew).length}}</q-badge>
          </q-tab>
          <q-tab v-if="loggedIn?teamMember:false" icon="star"  @click="$router.push('/admin')">
            <q-badge color="red" v-if="Object.keys(userReports).length" floating>{{Object.keys(userReports).length}}</q-badge>
            <q-badge color="amber" style="margin-top:20px" v-if="Object.keys(userFeedback).length" floating>{{Object.keys(userFeedback).length}}</q-badge>
          </q-tab>
      </q-tabs>
    </q-footer>

      <q-dialog v-model="myMarkersDialog">
        <my-markers/>
      </q-dialog>

      <q-dialog v-model="checkMarkersDialog" >
        <check-markers style="min-width:50%;max-width:500px"/>
      </q-dialog>

      <q-dialog v-model="feedbackDialog">
      <feedback-dialog @close="feedbackDialog=false"/>
    </q-dialog>

   
  </q-layout>
</template>

<script>
import { mapState, mapActions, mapGetters } from 'vuex'
import mixinGeneral from 'src/mixins/mixin-general.js'
import meta from 'src/utils/meta.js'

export default {
  meta,
  mixins: [mixinGeneral],
  
  data () {
    return {
      rightDrawerOpen: false,
      myMarkersDialog:false,
      checkMarkersDialog:false,
      feedbackDialog:false,
      tab:'',
        
     
      sections : {
        // {title: 'Donate', icon: 'fas fa-hand-holding-heart', to:'/donate',apple:true},
        "collections": {title: 'Collections', icon: 'collections', to:'/collections'},
        "resources":{title: 'Resources', icon: 'fas fa-rocket', to:'/resources'},
        "gear":{title: 'Gear', icon: 'fas fa-bicycle', to:'/gear'},
      },
    }
  },
  components: { 
		'feedback-dialog' : require('components/Shared/FeedbackDialog.vue').default,
    'drawer-right': require('src/layouts/DrawerRight.vue').default,
		'my-markers' : require('components/Marker/MyMarkers.vue').default,
		'check-markers' : require('components/Marker/CheckMarkers.vue').default,
    'map-all': require('components/Map/Map.vue').default,
  },
  computed:{
    ...mapState('auth',['loggedIn']),
		...mapGetters('chat', ['unreadchatlistnew']),
    ...mapState('admin', ['userFeedback','userReports']),

    otheruserId () {
      		return this.$route.params.otherUserId
    },
    otherUserDetails () {
			if (this.otheruserId) {
				return this.users[this.otherUserId]
			}
		},

    title() {
      let currentPath = this.$route.fullPath
      if (currentPath == '/'){
        return 'Cycle Planet'
      } 
      else if (currentPath.includes('/country')) {
        return this.$route.params.countryKey
      }
      else{
        return 'Cycle Planet'
      }
    },

  },
  methods:{
    ...mapActions('auth',['logoutUser']),
  
    logoutUserMethod(){
      this.logoutUser()
    },
    

  },
  mounted() {
    if(this.isLoggedIn){
      this.updateAppVersion()
      if(this.$route.fullPath === '/'){
        this.$router.push('/map')
      }
    }
  },
  meta() {
		return{
			title: 'Cycle Planet',
			titleTemplate: title => `${title} | The bicycle touring guide`,

			// meta tags
			meta: {
				description: { name: 'description', content: 'Page 1' },
				keywords: { name: 'keywords', content: 'bicycle, bike, cycle, cycling, touring, visa, border, safe, information' },
				equiv: { 'http-equiv': 'Content-Type', content: 'text/html; charset=UTF-8' },
				// note: for Open Graph type metadata you will need to use SSR, to ensure page is rendered by the server
				ogTitle:  {
					name: 'og:title',
					// optional; similar to titleTemplate, but allows templating with other meta properties
					template (ogTitle) {
					return `${ogTitle}`
					}
				}
			},

			// CSS tags
			link: {
			material: { rel: 'stylesheet', href: 'https://fonts.googleapis.com/icon?family=Material+Icons' }
			},

			// JS tags
			script: {
			ldJson: {
				type: 'application/ld+json',
				innerHTML: `{ "@context": "http://schema.org" }`
			}
			},

			// <html> attributes
			htmlAttr: {
			'xmlns:cc': 'http://creativecommons.org/ns#', // generates <html xmlns:cc="http://creativecommons.org/ns#">,
			empty: undefined // generates <html empty>
			},

			// <body> attributes
			bodyAttr: {
			'action-scope': 'xyz', // generates <body action-scope="xyz">
			empty: undefined // generates <body empty>
			},

			// <noscript> tags
			noscript: {
				default: 'This is content for browsers with no JS (or disabled JS)'
			}
		}
	}

    
     
  
}
</script>

<style >
@media screen and (min-width: 768px){
  .q-footer.layout{
    display: none;
  }
}



</style>


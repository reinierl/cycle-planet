<template>
<div class="q-ma-sm">

<div v-if="countryData&&pageReady&&checkCountry&&checkMarkersLoaded&&countriesAll">
	 <div v-if="countryData.countries">
       <q-banner inline-actions class="text-white bg-red " v-if="countryData.countries.settings.dangerous">
            <template v-slot:avatar>
              <q-icon name="warning" size="md" />
            </template>
			<div class="text-subtitle1">
			DANGER: Do not travel to {{countryKey}}. Check out the <span class="underline cursor-pointer " @click="openUrl('https://www.gov.uk/foreign-travel-advice')">U.K. travel advice</span> for the latest info.</div>
          </q-banner>
    </div>
  	<q-page v-if="countryData">
	<div v-if="countryData.countries">

		<div v-if="countryData.countries.zoom" class="row items-center">
			<q-avatar rounded style="width:auto" :size="isWebApp?'lg':'md'">
				<img  :src="('countryflags/Flag_of_'+countryKey+'.svg.png').split(' ').join('_')">
			</q-avatar>
			<p class="q-mt-md q-ml-sm" :class="isWebApp?'text-h4':'cp-h2'">{{countryKey}}</p>
			<!-- <q-icon class="q-mx-sm " :size="isWebApp?'md':'sm'" @click="loggedIn ? likeCountryHere((!country_likes[countryKey] ? 'liked' : 'disliked')) : showLoginDialog()" :name="!country_likes[countryKey] ? 'favorite_border' : 'favorite'" :class="!country_likes[countryKey] ? '' : 'text-red'"  /> -->
			<div class="absolute-right">
				<q-btn v-if="loggedIn?admin:false" flat round icon="settings" size="15px" @click.native="countrySettingsDialog=true"/>
			</div>
		</div>

		<div v-if="countryData.countries && mapsettings">
			<div v-if="countryData.countries.zoom">
				<l-map :options="screenwidthbig?{scrollWheelZoom:false}:{scrollWheelZoom:false, dragging:false, tap: false}" style="height: 350px" :zoom="zoom" :center="countryData.countries.location">
				<l-tile-layer :url="mapsettings.url" :attribution="mapsettings.attribution"></l-tile-layer>
					<map-marker :countryKey="countryKey"/>

					<l-control position="topleft"  class="q-ma-md">
                            <q-btn icon="add" dense class="row bg-white q-pa-xs" size="sm" @click="zoom++"/>
                            <q-btn icon="remove" dense class="row bg-white q-pa-xs q-mt-sm" size="sm" @click="zoom--"/>
                        </l-control>
				</l-map>
			</div>
		</div>
		</div>

	
		<!-- <div v-if="countryData.countries">
			<div v-if="countryData.countries.contactperson">
			<p class="cp-h2 q-mb-none">Contact person for {{countryKey}}</p>
			<div class="row">
				<div class="q-mx-xs" style="margin-right:-25px; " v-for="(user, userId) in countryData.countries.contactperson" :key="userId">
				<q-btn flat dense round size="sm" :to="'/user/'+userId">
					<q-avatar style="">
						<q-img :src="users[userId].imageurl" style="width:50px; "></q-img>
					</q-avatar>
					<q-tooltip content-class="">{{users[userId].fullname}}</q-tooltip>
				</q-btn>
				</div>
			</div>
			</div>
		</div> -->

	<div v-for="(refData,refKey,index) in refsextra" :key="index" v-if="countryData" >
		<div style="min-height:100px;margin-top:35px" v-if="!(refKey==='Embassy'&&isSchengenCountry)">
			<div v-if="refData.type==='single'">
				<single-item :refKey="refKey" :countryKey="countryKey" :data="countryData[refKey]"/>
			</div>

			<div v-if="refData.type==='summarylist'">
				<summary-item :refKey="refKey" :countryKey="countryKey" :data="countryData[refKey]"/>
			</div>

			<div v-if="refData.type==='postlist'">
				<post-item :refKey="refKey" :countryKey="countryKey"/>
			</div>

			<div v-if="refData.type==='tilelist'">
				<tile-item :refKey="refKey" :countryKey="countryKey"/>
			</div>

		</div>
	</div>


	<q-dialog :maximized="!isWebApp" v-model="countrySettingsDialog" >
		<!-- <page-item/> -->
		<edit-country-settings
		:countryKey="countryKey"
		@close="countrySettingsDialog = false" 
		/>
	</q-dialog>

	
  	</q-page>
</div>
<div v-else>
	<loading-page/>
</div>
</div>

</template>

<script>
import { mapState, mapActions, mapGetters  } from 'vuex'
import mixinGeneral from 'src/mixins/mixin-general.js'
import meta from 'src/utils/meta.js'
import { LMap, LTileLayer, LControl, LMarker,LIcon, LPopup, LFeatureGroup } from 'vue2-leaflet'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'
import { latLng, Icon } from 'leaflet';
delete Icon.Default.prototype._getIconUrl;
Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
});
	export default {
        props:['countryKey'],
		meta,
        mixins: [mixinGeneral],
        components:{
		LMap,
		LTileLayer,
		LControl,
		LMarker,
		LIcon,
		LPopup,
		'single-item': require('components/Country/SingleItem.vue').default,
		'summary-item': require('components/Country/SummaryItem.vue').default,
		'tile-item': require('components/Country/TileItem.vue').default,
		'post-item': require('components/Country/PostItem.vue').default,
	    'map-marker' : require('components/Shared/MapMarker.vue').default,
		'edit-country-settings': require('components/Country/Modals/EditCountrySettings.vue').default,
	},
	
	data() {
	  	return {
			loading: true,
			pageReady:false,
			zoom:6,
			
			countrySettingsDialog:false,
			title: this.$route.params.countryKey,
			loadedLandmarkers:false,
			// metaTags: {
			// 	// description: this.$route.params.countryKey,
			// 	' | Cycle Planet',
			// 	// url: 'https://cycleplanet.org/country/'+this.$route.params.countryKey,
			// 	// image: 'absolute url to share image'
            // }
		
	  	}
	  },
	 
	  computed: {
		...mapState('country', ['singleItemData','countryData']),
		checkMarkersLoaded(){
			if(!this.loadedLandmarkers){
				this.loadedLandmarkers=false
				this.refreshData()
				return false
			}else{
				return true
			}
		},
		
		checkCountry(){
			console.log('checkCountry',this.countryData);
			if(this.countryData.countries){
				if(this.countryKey===this.countryData.countries.name){
					console.log('checkCountry',true);
					return true
				}else{
					console.log('checkCountry',false);
					this.pageReady=false
					this.refreshData()
					this.pageReady=true 
					return true
				}
			}
			
		},
		isSchengenCountry(){
			console.log('isSchengenCountry 1',this.countryKey);
			console.log('isSchengenCountry 2',Object.keys(this.countriesAll).length);
			if(Object.keys(this.countriesAll).length){
				if(this.countriesAll[this.countryKey].schengen){
					return true
				}else{
					return false
				}
			}else{
				return false
			}
			
        },
		  
		
	  },
	 
	methods: {
		...mapActions('countries',['likeCountry']),
		...mapActions('country', ['destroyData','firebaseGetCountryDataNew']),
		...mapActions('markers', ['destroyMapData']),
		likeCountryHere(type){
			this.likeCountry({
				type:type,
				countryKey:this.countryKey,
			})
		},
	
		refreshData(){
			console.log('refreshData 1');
			this.destroyMapData().then(()=>{
			console.log('refreshData 2');
			})
			console.log('refreshData 3');
			this.destroyData().then(() => {
			console.log('refreshData 4');
				this.firebaseGetCountryDataNew(this.countryKey);
			})
		}
	},
	
    mounted() {
		if(this.landMarkers){
			this.loadedLandmarkers=true
		}
		console.log('mounted pagecountry started 1');
        if(this.countryKey){
		console.log('mounted pagecountry started 2');
			this.refreshData()
			this.pageReady=true 
	  	 }
		if(!this.loadedPosts){
			console.log('created userprofile - about tab - not loaded posts yet');
    		this.getPosts()
		}
		this.zoom=this.countries[this.countryKey].zoom
	},
	destroyed() {//call a method when page is left
	console.log('destroyed CountryPage');
		this.pageReady=false
		this.destroyMapData();
    },
	meta() {
		return{
			// sets document title
			title: this.title,
			// optional; sets final title as "Index Page - My Website", useful for multiple level meta
			titleTemplate: title => `${title} | Cycle Planet`,
			// meta tags
			meta: {
				description: { name: 'description', content: 'Page 1' },
				keywords: { name: 'keywords', content: `${this.title}, bicycle, bike, cycle, cycling, touring, visa, border, safe, information` },
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
<style>
.q-card{
	padding:8px;
	margin-bottom: 16px;
}
img{
	width: 100%;
}
</style>
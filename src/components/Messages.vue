<template>
	<div class="application">
		<auth-component></auth-component>	
		<div class="head">
			<h3>{{ userInformation.name }}</h3>
			<h3>{{ date.getDate() }} {{date.toLocaleString('en-us', { month: "short" })}}, {{date.getFullYear()}}</h3>
		</div>

		<hr>
		<div class="messages">

			<div class="message" v-for="message in messages">
				<div class="message-head">
					<span>{{ message.name }}</span>
					<img :src="message.photo" alt="">
				</div>
				<div class="message-box">
					<span class="date">{{ message.date }}</span>
					<div class="content"><p>{{ message.message }}</p></div>
				</div>

			</div>
		</div>


		<div class="footer">
			<div class="footer-logo" @click="addMessage">
				<img src="../imgs/plus.png" height="30" width="30" alt="">
			</div>
			<input type="text" v-model="newMessage" @keyup="addMessage">
		</div>

	</div>
</template>


<script>

	import Vue from 'vue';
	import Auth from './Auth.vue';
	var firebase = require('firebase');
	var VueFire = require('vuefire');

	var config = {
		apiKey: "AIzaSyDTQQvHD6GlCyi1YqRhNqWYVH2YL-95YSY",
		authDomain: "vuechat-aa77f.firebaseapp.com",
		databaseURL: "https://vuechat-aa77f.firebaseio.com",
		storageBucket: "vuechat-aa77f.appspot.com",
		messagingSenderId: "632804180168"
	};

	var firebaseApp = firebase.initializeApp(config);
	var db = firebaseApp.database();



	Vue.use(VueFire);

	Vue.component('auth-component', {
		mounted: function() {
			console.log(1)
			this.$on('Auth', this.removeFromList);
		}
	})
	export default {

		data(){
			return {
				messages: [],
				newMessage: '',
				date: '',
				database: '',
				token: '',
				userInformation: {}
			}
		},

		mounted: function() {
			var that = this;
			this.$on('Auth-user', function	(user, token) {
				db.ref().child('users').on("child_added", function(snap) {
					if (snap.val().id === user.uid){
						document.cookie = token;		
						that.userInformation = snap.val();
					}
				});

				setTimeout(function() {

					if (!document.cookie) {
						db.ref().child('users').push({
							'name': user.displayName,
							'email' : user.email,
							'photo': user.photoURL,
							'id': user.uid,
							'refreshToken': user.refreshToken,
							'token': token
						});
					}

				}, 1000);



			});
		},

		methods:{

			getData: function() {
				var that = this;
				db.ref().child('messages').on("child_added", function(snap) {
					that.messages.push(snap.val());
				});
				setTimeout(this.scrollBottom, 2000);
			},


			addMessage: function(event) {
				if ((event.keyCode === 13 || event.button === 0) && this.newMessage !== ''){		
					var date = new Date();	
					this.writeData({
						'name': this.userInformation.name.split(" ")[0],
						'date': date.getHours() + ':' + date.getMinutes() + ', ' + date.getDate() + ' ' + date.toLocaleString('en-us', { month: "short" }),
						'message': this.newMessage,
						'photo': this.userInformation.photo
					});
					this.newMessage = '';

					setTimeout(this.scrollBottom, 5);
				}
			},

			writeData: function	(obj) {
				db.ref().child('messages').push(obj);
			},

			scrollBottom: function() {
				var element = document.getElementsByClassName('messages')[0];
				element.scrollTop = element.scrollHeight;
			}
		},

		created() {
			this.getData();
			this.date = new Date();
			this.database = firebase.database();
			document.cookie	= '';


		},

		components :{
			'auth-component': Auth
		}

	}
</script>



<style lang="sass">

.application
	width: 850px
	height: 650px
	margin: 100px auto
	background: #5B585E
	box-shadow: 0 0 10px 5px rgba(black, 0.3)
	user-select: none

	.head 
		display: flex	
		justify-content: space-between
		padding: 10px 50px
		height: 10%
		color: white

	.footer-logo
		display: flex
		justify-content: center
		align-items: center

	.messages
		width: 90%
		height: 70%
		margin: 0 auto
		display: flex
		overflow-y: auto
		align-items: flex-end
		flex-direction: column

	.message
		display: flex
		padding: 10px
		color: white
		min-width: 60%
		min-height: 120px
		align-items: center

		span
			margin-bottom: 5px

	.message-head
		text-align: center
		width: 80px
		height: 100px
		margin-right: 10px
		display: flex
		justify-content: center
		align-items: center
		flex-direction: column

		img
			width: 50px
			height: 50px
			border-radius: 50%

	.message-box
		display: flex
		flex-direction: column
		justify-content: center
		width: 100%

	.date
		font-size: 14px
		background: #D46A6A
		font-style: italic
		width: 100px
		text-align: center
		margin-bottom: 5px
		border-radius: 5px

	.content
		font-size: 14px
		background: #D46A6A
		text-align: center
		height: 60px
		border-radius: 4px
		padding: 5px
		display: flex
		justify-content: center
		align-items: center

		p
			margin: 0
			padding: 0
	hr
		width: 75%
		border: none
		color: teal
		background-color: teal
		height: 2px

	.footer
		display: flex
		width: 90%
		margin: 10px auto
		height: 10%
		box-shadow: 0 0 10px 5px rgba(black, 0.3)


	.footer-logo
		width: 70px
		background: #D46A6A

	input
		width: 100%
		background: teal
		outline: none
		border: none
		text-align: center
		color: white
		font-size: 16px



	::-webkit-scrollbar 
	  width: 15px

	::-webkit-scrollbar-thumb 
	  background: teal

	::-webkit-scrollbar-thumb:window-inactive 
	  background: teal


</style>
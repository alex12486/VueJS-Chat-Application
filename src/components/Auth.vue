
<template>

</template>



<script>

	var firebase = require('firebase');




	export default {
		data(){
			return {

			}
		},


		methods: {

		},

		created(){

			var provider = new firebase.auth.GoogleAuthProvider();
			var that = this;

			firebase.auth().signInWithPopup(provider).then(function(result) {

				var token = result.credential.accessToken;
				var user = result.user;

				that.$parent.$emit('Auth-user', user, token);

			}).catch(function(error) {

				var errorCode = error.code;
				var errorMessage = error.message;
				var email = error.email;
				var credential = error.credential;

				console.log('errorCode: '+ errorCode + '\nerrorMessage: ' + errorMessage + '\nemail: ' + email + '\ncredential: ' + credential);

			});
		}
	}
</script>


<style></style>
<template>
  <div>
    <div class="post-up" @click.self="$emit('close-post')">
      <div class="post-up__content box-border h-auto bg-white z-30 rounded fixed my-0">
        <div class="back-btn p-2 border border-solid　border rounded">
          <img src="/images/back.svg" class="back-point h-5 cursor-pointer" @click.self="$emit('close-post')">
        </div>
        <div class="user my-2 ml-4 flex">
          <div class="avatar mr-3 border rounded-full border-solid border-black">
            <nuxt-link :to="`/users/${user.id}`" v-if="isAuthenticated">
              <img :src="user.photoURL" class="user-image w-8 h-8 rounded-full" alt="">
            </nuxt-link>
            <span v-if="!isAuthenticated">
              <img :src="user.photoURL" class="user-image w-8 h-8 rounded-full" alt="">
            </span>
          </div>
          <div class="user-name leading-loose text-sm">
            <nuxt-link :to="`/users/${user.id}`" v-if="isAuthenticated">
              <p class="font-bold">{{ user.displayName }}</p>
            </nuxt-link>
            <p v-if="!isAuthenticated" class="font-bold">{{ user.displayName }}</p>
          </div>
        </div>
        <a class="post-desk__shop font-bold ml-4 break-all" :href="postDetail.restaurantsUrl" target="_blank">{{ postDetail.restaurantsName }}</a>
        <div class="post-desk__img">
          <img :src="postDetail.image" alt="">
        </div>
        <div class="message my-2 ml-4 flex">
          <img v-if="beLiked" src='/images/heart_active.svg' @click="unlike" class="w-6 mr-3">
          <img v-else src='/images/heart.svg' @click="like"  class="w-6 mr-3">
          <p>{{ likeCount }}</p>
        </div>
        <div class="message mx-4 text-sm">
          <nuxt-link :to="`/users/${user.uid}`" v-if="isAuthenticated">
            <span class="font-bold">{{ user.displayName }}</span>
          </nuxt-link>
          <span  v-if="!isAuthenticated" class="font-bold">{{ user.displayName }}</span>
          <span>{{ postDetail.text }}</span>
        </div>
        <span class="message mx-4 text-sm text-gray-600">コメント</span>
        <div class="message mx-4 text-sm">
          <div v-for="(comment, index) in comments" :key="index" :comment="comment">
            <span class="font-bold">{{ comment.userName }}</span>
            <span class="break-all">{{ comment.comment }}</span>
          </div>
        </div>
        <div class="message mx-4 text-sm flex justify-between border-t border-gray-30 p-3">
          <textarea
              class="p-3 w-4/5 outline-none resize-none"
              :rows="1"
              :cols="40"
              placeholder="コメントを追加"
              v-model="postComment"
          />
          <button class="post-desk__post-cmt font-semibold text-blue-500 " @click="setComment">投稿する</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { db } from '~/plugins/firebase'

export default {
  props: ['postDetail'],
  data () {
    return {
       user: {
        id: '',
        displayName: '',
        photoURL: ''
      },
      likeCount: 0,
      beLiked: false,
      comments: [],
      postComment: null
    }
  },
  async mounted () {
    const userId = this.postDetail.userId
    const doc = await db.collection('users').doc(userId).get()
    this.user = { ...doc.data(), id: userId }
  
    this.$emit('likeSnap')
    this.commentRef = db.collection('posts').doc(this.postDetail.createdAt).collection('comments')
    this.checkComment()
  },
  methods: {
    like () {
      this.$emit('like')
      this.beLiked = true
    },
    unlike () {
      this.$emit('unlike')
      this.beLiked = false
    },
    async setComment () {
      let time = new Date().getTime()

      await this.commentRef.add({
        comment: this.postComment,
        userName: this.currentUser.displayName,
        userId: this.currentUser.uid,
        createdAt: time + ''
      })
      this.postComment = null
    },
    async checkComment () {
      await this.commentRef.orderBy('createdAt').onSnapshot((snapshot) => {
        snapshot.docChanges().forEach((change) => {
          const doc = change.doc
          this.comments.push(doc.data())
        })
      })
    }
  },
  computed: {
    currentUser () {
      return this.$store.state.user
    },
    isAuthenticated () {
      return this.$store.getters.isAuthenticated
    }
  }
}
</script>

<style scoped>
.post-up {
  display: flex;
  align-items: center;
  justify-content: center;
  position: fixed;
  z-index: 20;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0,0,0,.8);
  opacity: 1;
}

.post-up__content {
  width: 25%;
}

.post-desk__img img{
  box-sizing: border-box;
  width: 100%;
  max-width: 100%;
  max-height:500px;
  height : auto;
}

@media screen and (max-width: 1360px) {
  .post-up__content {
    width: 50%;
  }

  .post-desk__img img{
    max-height:450px;
  }
}

@media screen and (max-width: 800px) {
  .post-desk__img img{
    max-height:400px;
  }
}

@media screen and (max-width: 480px) {

.back-btn {
  padding: 0.25rem;
}

.back-point {
  height: 1rem;
}

 .post-desk__img img{
    max-height:400px;
  }
}

@media screen and (max-width: 480px) {
  .post-desk__img img{
    max-height:400px;
  }
}

@media screen and (max-width: 320px) {
  .post-desk__img img{
    max-height:280px;
  }
}

</style>

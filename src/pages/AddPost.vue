<template>
  <div>
    <!-- Form Add Files -->
    <q-form class="formBaseUpload">
      <!-- Add File -->
      <input
        type="file"
        @change="addFiles($event)"
        multiple="multiple"
        class="uploader"
        ref="uploader"
      />

      <!-- Description -->
      <q-input
        v-model="descriptionPost"
        type="textarea"
        class="q-mt-md"
        standout="bg-grey-6 text-white"
        label="Write a caption ..."
      />
      <q-btn
        class="full-width q-mt-md"
        :color="
          descriptionPost.length <= 0 || images.length <= 0
            ? 'light-blue-1'
            : 'light-blue'
        "
        dense
        label="Upload"
        :disabled="descriptionPost.length <= 0 || images.length <= 0"
        @click="uploadFiles"
      />
    </q-form>
    <div class="separator">
      <q-separator size="1px" color="grey-5" />
    </div>
    <!-- Files -->
    <div
      v-for="(img, index) in imagesOfPosts"
      :key="index"
      class="filesBase relative-position"
    >
      <q-img :ratio="1" :src="img" class="fileImg"> </q-img>
      <q-btn
        icon="cancel"
        class="absolute-top-right q-ma-lg"
        color="white"
        style="z-index: 9"
        @click="eraseImage(index)"
        dense
        flat
      />
    </div>
  </div>
</template>
<script>
import { firebaseAuth, firebaseStorage, firebaseDb } from "boot/firebase";
import { Loading, Notify, uid } from "quasar";
export default {
  data() {
    return {
      imagesOfPosts: [],
      descriptionPost: "",
      images: [],
    };
  },
  methods: {
    addFiles(file) {
      Object.values(file.target.files).forEach((file) => {
        if (
          file.type !== "image/png" &&
          file.type !== "image/jpeg" &&
          file.type !== "image/jpg"
        ) {
          Notify.create({
            message: "This uploader only accepts png, jpeg and jpg",
            color: "negative",
            icon: "error",
          });
        } else {
          let newFile = URL.createObjectURL(file);
          this.imagesOfPosts.push(newFile);
          this.images.push(file);
        }
      });
    },
    uploadFiles() {
      Loading.show({
        message: "Uploading your post...",
      });
      let descriptionOfPost = "";
      descriptionOfPost = this.descriptionPost;
      let imagesUploaded = [];
      const uniqueId = uid();
      this.images.forEach((file) => {
        let randomId = uid();
        const imagesStirageRef = firebaseStorage
          .ref(firebaseAuth.currentUser.uid + "/posts/" + randomId)
          .put(file)
          .then((data) => {
            firebaseStorage
              .ref(firebaseAuth.currentUser.uid + "/posts/" + randomId)
              .getDownloadURL()
              .then((url) => {
                imagesUploaded.push(url);
                // Create post in own user
                const userPosts = firebaseDb
                  .ref()
                  .child(
                    "toneygram/users/" +
                      firebaseAuth.currentUser.uid +
                      "/posts/" +
                      uniqueId
                  );
                let dateOfPost =
                  new Date().getDate() +
                  "/" +
                  (new Date().getMonth() + 1) +
                  "/" +
                  new Date().getFullYear();

                userPosts.set({
                  idPost: uniqueId,
                  imagesUploaded,
                  userInfo: {
                    userName: firebaseAuth.currentUser.displayName,
                    userId: firebaseAuth.currentUser.uid,
                    userImg: firebaseAuth.currentUser.photoURL,
                  },
                  description: descriptionOfPost,
                  dateOfPost: dateOfPost,
                  fullD: new Date().getTime(),
                  scrollIndex: 0,
                });
                // Create post outside Users
                const userPostOutSide = firebaseDb.ref(
                  "toneygram/posts/" +
                    firebaseAuth.currentUser.uid +
                    "/" +
                    uniqueId
                );
                userPostOutSide.set({
                  idPost: uniqueId,
                  imagesUploaded,
                  userInfo: {
                    userName: firebaseAuth.currentUser.displayName,
                    userId: firebaseAuth.currentUser.uid,
                    userImg: firebaseAuth.currentUser.photoURL,
                  },
                  description: descriptionOfPost,
                  dateOfPost: dateOfPost,
                  fullD: new Date().getTime(),
                  scrollIndex: 0,
                });
                Loading.hide();
                Notify.create({
                  color: "positive",
                  message: "Your post has been succesfully uploaded",
                  icon: "done",
                });
                this.imagesOfPosts = [];
                this.images = [];
                this.$refs.uploader.value = "";
                this.descriptionPost = "";
              })
              .catch((error) => {
                Loading.hide();
                Notify.create({
                  color: "negative",
                  message:
                    "There was a problem trying to upload your post, please, try again.",
                  icon: "error",
                });
              });
          })
          .catch((error) => {
            Notify.create({
              color: "negative",
              message:
                "There was a problem trying to upload your post, please, try again.",
              icon: "error",
            });
            Loading.hide();
          });
      });
    },
    eraseImage(index) {
      this.imagesOfPosts.splice(index, 1);
      this.images.splice(index, 1);
      this.$refs.uploader.value = "";
    },
  },
};
</script>
<style lang="scss">
//iPhone
@media (max-width: 480px) {
  .formBaseUpload {
    padding: 1rem;
  }
  .filesBase {
    padding: 1rem;
    width: 100%;
    height: 20rem;
  }
  .fileImg {
    width: 100%;
    height: 100%;
  }
  .separator {
    padding: 0 1rem;
  }
}
//Tablet
@media (min-width: 480px) {
  .formBaseUpload {
    padding: 1rem;
    max-width: 500px;
    margin: auto;
  }
  .filesBase {
    padding: 1rem;
    width: 100%;
    height: 20rem;
    max-width: 500px;
    margin: auto;
  }
  .fileImg {
    width: 100%;
    height: 100%;
  }
  .separator {
    padding: 0 1rem;
    max-width: 500px;
    margin: auto;
  }
}
//Desktop
@media (min-width: 768px) {
  .formBaseUpload {
    padding: 1rem;
    max-width: 500px;
    margin: auto;
  }
  .filesBase {
    padding: 1rem;
    width: 100%;
    height: 20rem;
    max-width: 500px;
    margin: auto;
  }
  .fileImg {
    width: 100%;
    height: 100%;
  }
  .separator {
    padding: 0 1rem;
    max-width: 500px;
    margin: auto;
  }
}

.avatar-upload {
  position: relative;
  max-width: 205px;
  margin: auto;
  .avatar-edit {
    position: absolute;
    right: 12px;
    z-index: 1;
    top: 10px;
    input {
      display: none;
      + label {
        display: inline-block;
        width: 34px;
        height: 34px;
        margin-bottom: 0;
        border-radius: 100%;
        background: #ffffff;
        border: 1px solid transparent;
        box-shadow: 0px 2px 4px 0px rgba(0, 0, 0, 0.12);
        cursor: pointer;
        font-weight: normal;
        transition: all 0.2s ease-in-out;
        &:hover {
          background: #f1f1f1;
          border-color: #d6d6d6;
        }
        &:after {
          content: "\e853";
          color: #757575;
          position: absolute;
          top: 10px;
          left: 0;
          right: 0;
          text-align: center;
          margin: auto;
        }
      }
    }
  }
  .avatar-preview {
    width: 192px;
    height: 192px;
    position: relative;
    border-radius: 100%;
    border: 6px solid #f8f8f8;
    box-shadow: 0px 2px 4px 0px rgba(0, 0, 0, 0.1);
    > div {
      width: 100%;
      height: 100%;
      border-radius: 100%;
      background-size: cover;
      background-repeat: no-repeat;
      background-position: center;
    }
  }
}
</style>

<template>
  <v-container grid-list-x1 fluid>
    <v-layout row wrap>
      <v-flex xs11>
        <v-carousel>
          <v-carousel-item v-for="item in items" :key="item.photo" :src="imgaddress(item.photo)"></v-carousel-item>
        </v-carousel>
      </v-flex>
      <v-flex xs12 offset-xs5>
        <v-btn flat @click="handlerCilik">
          <v-icon color="primary">cloud_upload</v-icon>&nbsp;上传图片
        </v-btn>

        <input
          type="file"
          id="upload"
          ref="upload"
          @change="changeimg"
          accept=".jpg, .jpeg, .png"
          v-show="false"
        />
        <v-dialog v-model="dialog" max-width="290">
          <v-card>
            <v-card-title class="headline">上传成功</v-card-title>

            <v-card-text>{{this.fileinfo}}</v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>

              <v-btn color="green darken-1" flat="flat" @click="dialog = false">确认</v-btn>

              <v-btn color="green darken-1" flat="flat" @click="dialog = false">关闭</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-flex>
      <v-flex xs12 style="margin-top:50px">
        <v-select :items="cities" label="城市" item-text="name" item-value="id" v-model="city_id"></v-select>
      </v-flex>

      <v-flex lg4 sm6 xs12 v-for="(item,i) in items" :key="item.id" style="padding:12px">
        <!-- <v-card  >
          <v-card-text>
          <v-text-field v-model="items[i].photo" label="轮播图" placeholder="轮播图地址"></v-text-field>
          <v-select :items="ariticles" label="文章" placeholder="文章" v-model="items[i].article_id" item-text="title" item-value="id"></v-select>
          </v-card-text>
                    <v-card-actions>
              <v-spacer></v-spacer>

              <v-btn color="red darken-1" dark flat="flat" @click="deletecard">删除<v-icon dark right>delete</v-icon></v-btn>
            </v-card-actions>
        </v-card>-->
        <!-- :photo.sync="item[0].photo" -->
        <lantern-card
          :index="i"
          :city_id="city_id"
          :article_id.sync="items[i].article_id"
          :photo_address.sync="items[i].photo"
          :city_list="cities"
          @delectcard="deletecard"
        ></lantern-card>
      </v-flex>
      <v-flex xs12>
        <v-btn color="orange darken-2" dark @click="addlantern">
          添加
          <v-icon dark right>add</v-icon>
        </v-btn>
        <v-btn color="primary" dark @click="savelanterns">
          确认
          <v-icon dark right>fa fas fa-paper-plane</v-icon>
        </v-btn>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import { Lantern, GetCities } from "@/api/toGet";
import { SetLantern, Upload2 } from "@/api/toPost";
import LanternCard from "@/components/LanternCard.vue";
export default {
  data() {
    return {
      cities: [],
      items: [],
      city_id: 14,
      fileinfo: "",
      dialog: false,
      ariticles: [],
      cnt:1
    };
  },
  components: {
    LanternCard
  },
  methods: {
    getcity() {
      GetCities().then(res => {
        this.cities = res.data;
      });
    },
    getlantern() {
      Lantern({ city_id: this.city_id }).then(res => {
        this.items = res.data;
      });
    },
    imgaddress(image) {
      if (!image) return "";
      if (image.toString().startsWith("http")) {
        return image;
      } else {
        return process.env.VUE_APP_CDN + image;
      }
    },
    addlantern() {
      this.items.push({ article_id: 0, photo: "", id: - (this.cnt++) });
    },
    savelanterns() {
      let datas = {
        city_id: this.city_id,
        lantern_list: this.items
      };
      SetLantern(datas)
        .then(res => {
          if (res.code == 200) {
            this.$message({
              message: "保存成功",
              type: "success"
            });
          }
        })
        .catch(error => {
          console.log(error);
        });
    },
    deletecard(index) {
      this.items.splice(index, 1);
    },
    handlerCilik() {
      let uploadbtn = this.$refs.upload;
      uploadbtn.click();
    },
    async changeimg(e) {
      let formData = new FormData();
      formData.append("file", e.target.files[0]);
      Upload2(formData)
        .then(res => {
          this.fileinfo = res.data;
          this.dialog = true;
        })
        .catch(error => {
          this.$message({
            message: "上传异常",
            type: "error"
          });
          console.log(error);
        });
    }
  },
  created() {
    this.getcity();
    this.getlantern();

  },
  watch: {
    city_id: function() {
      this.getlantern();
    }
  }
};
</script>